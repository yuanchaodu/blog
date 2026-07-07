---
title: Linux 存储故障排查
section: IT
category: Linux
---

# Linux 存储故障排查

<img src="images/Linux.svg" width="300">

Linux 存储故障排查，最重要的不是记住很多命令，而是建立一条固定的排查链路：

**业务现象 → 系统日志 → I/O 状态 → 设备链路 → 磁盘健康 → RAID/LVM/多路径 → 文件系统**

可以把 Linux 存储看成一条水管：应用是水龙头，文件系统和 LVM 是管道，磁盘或存储阵列是水源。故障排查就是逐层判断“堵在哪里”。

## 一、先判断是哪一类故障

常见问题大致分为以下几类：

| 现象        | 常见原因                |
| --------- | ------------------- |
| 磁盘消失      | 硬盘、控制器、HBA、SAN 链路故障 |
| I/O 很慢    | 磁盘延迟、队列拥塞、阵列压力      |
| 文件系统只读    | 底层 I/O 错误或文件系统异常    |
| 空间不足      | 容量、inode 或快照占满      |
| I/O error | 磁盘、链路、RAID 或存储阵列故障  |
| 系统卡死      | I/O 长时间等待           |
| SAN 盘异常   | 多路径、交换机、HBA 或阵列问题   |

企业环境中，尤其是生产系统，建议先明确故障层级，不要一看到 `I/O error` 就直接认定是硬盘损坏。

## 二、第一步：保护现场

先记录时间、设备名和报错，不要急着重启。

```bash
date
uptime
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
df -hT
df -ih
```

生产系统中应避免直接执行这些高风险操作：

```bash
fsck
xfs_repair
mkfs
pvcreate
vgremove
lvremove
```

原因很简单：在设备关系没有确认前，“修复”有时比故障本身更危险。

## 三、第二步：先看内核日志

这是存储故障最重要的一步。

```bash
dmesg -T | tail -200
journalctl -k --since "-1 hour"
```

重点搜索：

```bash
dmesg -T | grep -Ei \
"error|fail|timeout|reset|I/O|blk|scsi|nvme|xfs|ext4"
```

常见关键词可以这样理解：

| 日志                   | 可能方向        |
| -------------------- | ----------- |
| `I/O error`          | 底层设备或链路     |
| `Buffer I/O error`   | 块设备访问失败     |
| `timing out command` | 设备响应超时      |
| `reset`              | 控制器或链路重置    |
| `EXT4-fs error`      | EXT4 文件系统   |
| `XFS ... corruption` | XFS 文件系统    |
| `nvme timeout`       | NVMe 设备或控制器 |
| `rejecting I/O`      | 设备已经不可用     |

一个重要原则是：**先找最早出现的错误，不要只看最后一条。** 后面的文件系统错误可能只是底层故障引起的连锁反应。

## 四、第三步：确认设备关系

现代 Linux 存储通常不是“一个分区对应一个硬盘”，而是多层结构：

```text
应用
 ↓
文件系统
 ↓
LVM
 ↓
Multipath / RAID
 ↓
SCSI / NVMe
 ↓
物理磁盘或存储阵列
```

建议执行：

```bash
lsblk -f
lsblk -o NAME,MAJ:MIN,SIZE,TYPE,FSTYPE,MOUNTPOINTS
findmnt
blkid
```

如果使用 LVM：

```bash
pvs
vgs
lvs -a -o +devices
```

这里的目标只有一个：**把报错设备映射到真正的底层设备。**

例如：

```text
/data
 ↓
/dev/mapper/vg_data-lv_data
 ↓
/dev/mapper/mpatha
 ↓
/dev/sdb + /dev/sdc
 ↓
SAN 存储
```

此时 `/data` 出错，并不代表文件系统本身有问题。

## 五、第四步：判断是不是性能问题

使用：

```bash
iostat -xz 1
```

重点看：

* `r/s`、`w/s`：读写次数；
* `r_await`、`w_await`：读写等待时间；
* `aqu-sz`：I/O 队列长度；
* `%util`：设备忙碌程度。

不要单独根据 `%util=100%` 判断磁盘故障。对于 SSD、NVMe、SAN 和虚拟磁盘，单一指标可能产生误判。应同时结合延迟、队列、吞吐量和业务响应。

如果出现：

```text
await 持续升高
队列持续增长
业务响应越来越慢
```

通常说明 I/O 已经出现拥塞。

## 六、第五步：检查磁盘健康

对于 SATA/SAS/SSD，可以使用 `smartctl`：

```bash
smartctl -a /dev/sda
```

对于 NVMe：

```bash
smartctl -a /dev/nvme0
```

重点关注：

```text
SMART overall-health
Reallocated Sector
Pending Sector
Uncorrectable Error
Media Errors
Critical Warning
Temperature
Percentage Used
```

`smartctl` 和 `smartd` 是 [smartmontools](https://www.smartmontools.org) 提供的主要工具，可监控 ATA、SCSI 和 NVMe 存储设备。([smartmontools.org][1])

需要注意：**SMART 正常不等于磁盘一定正常。** 存储故障也可能发生在 RAID 卡、HBA、线缆、背板、SAN 交换机或存储阵列。

## 七、第六步：SAN 和多路径检查

如果服务器连接外部存储，重点检查：

```bash
multipath -ll
systemctl status multipathd
journalctl -u multipathd
```

还可以查看：

```bash
multipathd show paths
multipathd show maps
```

正常情况下，一个存储 LUN 通常存在多条访问路径。DM Multipath 将多条物理 I/O 路径组织成一个逻辑设备，并可提供故障切换能力。([红帽文档][2])

例如：

```text
mpatha
├── sdb  active ready running
├── sdc  active ready running
├── sdd  active ready running
└── sde  active ready running
```

如果变成：

```text
sdb failed faulty
sdc failed faulty
```

排查方向应转向：

```text
服务器 HBA
  ↓
光纤链路
  ↓
SAN 交换机
  ↓
存储控制器
```

而不是直接修复文件系统。

## 八、推荐的标准排查顺序

我建议实际工作中固定使用下面这一套顺序：

1. **确认业务现象**：慢、卡、只读、设备消失还是 I/O error。
2. **记录故障时间**：后续日志必须围绕这个时间点分析。
3. **检查内核日志**：优先找最早的底层错误。
4. **确认设备拓扑**：文件系统 → LVM → Multipath/RAID → 物理设备。
5. **检查 I/O 性能**：判断是故障还是拥塞。
6. **检查设备健康**：SMART、NVMe、RAID 状态。
7. **检查存储链路**：HBA、Multipath、SAN、阵列。
8. **最后检查文件系统**：确认底层稳定后再考虑修复。

这套思路的核心可以概括成一句话：

> **先查底层，再查上层；先收集证据，再进行修复。**

[1]: https://www.smartmontools.org "smartmontools"
[2]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_device_mapper_multipath/overview-of-device-mapper-multipathing_configuring-device-mapper-multipath "Chapter 1. Overview of device mapper multipathing"