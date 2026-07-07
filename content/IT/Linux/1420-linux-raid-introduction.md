---
title: Linux raid 基础
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnnXd
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/142'
---

# Linux raid 基础

<img src="images/Linux.svg" width="300">

# Linux RAID 基础

Linux RAID 是服务器存储管理中的基础技术。可以先把它理解为：

> **把多块物理磁盘组合成一块逻辑磁盘，以提高性能、容量或可靠性。**

在 Linux 中，最常见的是**软件 RAID**。它由 Linux 内核的 **MD（Multiple Devices）** 模块实现，通常使用 `mdadm` 命令管理。Linux 当前支持 RAID 0、1、4、5、6、10 等类型。([Linux内核文档][1])

官方资料：[Linux 内核 RAID 文档](https://docs.kernel.org/admin-guide/md.html)、[mdadm 命令手册](https://man7.org/linux/man-pages/man8/mdadm.8.html)。

## 一、RAID 到底解决什么问题？

假设服务器有多块硬盘：

```text
/dev/sdb
/dev/sdc
/dev/sdd
/dev/sde
```

不用 RAID 时，每块硬盘相互独立：

```text
硬盘1 → 单独使用
硬盘2 → 单独使用
硬盘3 → 单独使用
```

使用 RAID 后：

```text
/dev/sdb ─┐
/dev/sdc ─┼──→ RAID → /dev/md0
/dev/sdd ─┤
/dev/sde ─┘
```

操作系统看到的是一个新的块设备，例如：

```text
/dev/md0
```

然后可以像普通磁盘一样使用：

```text
/dev/md0
   ↓
文件系统 ext4 / XFS
   ↓
挂载目录 /data
```

Linux 软件 RAID 本质上就是把多个真实块设备组合成一个虚拟块设备。([man7.org][2])

## 二、最重要的 RAID 级别

| 类型      | 最少磁盘 | 可用容量 |    允许损坏 | 主要特点      |
| ------- | ---: | ---: | ------: | --------- |
| RAID 0  |    2 | 100% |     0 块 | 速度快，无保护   |
| RAID 1  |    2 |  50% |  通常 1 块 | 简单可靠      |
| RAID 5  |    3 |  N-1 |     1 块 | 容量利用率较高   |
| RAID 6  |    4 |  N-2 |     2 块 | 安全性更高     |
| RAID 10 |    4 |  50% | 视损坏位置而定 | 性能和可靠性较均衡 |

### RAID 0：条带化

```text
数据 A → 磁盘1
数据 B → 磁盘2
数据 C → 磁盘1
数据 D → 磁盘2
```

两块磁盘同时工作，所以性能较好。

但问题也很明显：

```text
任何一块磁盘损坏
        ↓
整个 RAID 可能无法使用
```

因此，RAID 0 **没有数据冗余能力**。

### RAID 1：镜像

```text
          ┌→ 磁盘1：ABC
数据 ABC ─┤
          └→ 磁盘2：ABC
```

两块磁盘保存相同的数据。

例如：

```text
2 × 4 TB 磁盘
可用容量约为 4 TB
```

优点是结构简单、恢复方便。对于系统盘和重要的小型服务器，RAID 1 很常见。

### RAID 5：条带 + 单校验

```text
磁盘1    磁盘2    磁盘3
 A        B        P
 C        P        D
 P        E        F
```

其中 `P` 是校验信息。

如果一块磁盘损坏，可以利用其他磁盘的数据和校验信息恢复。RAID 5 至少需要 3 块磁盘，容量大约是：

```text
(N - 1) × 单块磁盘容量
```

例如：

```text
4 × 4 TB
≈ 12 TB 可用容量
```

### RAID 6：双校验

RAID 6 与 RAID 5 类似，但有两份校验信息。

```text
允许同时损坏两块磁盘
```

容量大约是：

```text
(N - 2) × 单块磁盘容量
```

例如：

```text
6 × 4 TB
≈ 16 TB 可用容量
```

对于大容量磁盘阵列，RAID 6 通常比 RAID 5 更稳妥。

### RAID 10：镜像 + 条带

RAID 10 可以简单理解为：

```text
先做镜像，再做条带
```

例如：

```text
磁盘1 ─┐
       ├─ RAID 1 ─┐
磁盘2 ─┘          │
                  ├─ RAID 0 → RAID 10
磁盘3 ─┐          │
       ├─ RAID 1 ─┘
磁盘4 ─┘
```

它兼顾性能和可靠性，但只能使用大约 50% 的原始容量。

## 三、Linux 软件 RAID 的核心组件

Linux RAID 主要涉及三个东西。

**第一，MD 驱动。** 它是 Linux 内核中的 RAID 功能模块，负责真正的数据读写和 RAID 逻辑。

**第二，`mdadm`。** 它是主要的管理工具，可以创建、查看、组装、扩容和维护 RAID。([man7.org][2])

**第三，`/dev/mdX`。** RAID 创建后通常表现为：

```text
/dev/md0
/dev/md1
/dev/md2
```

所以，一个典型结构是：

```text
物理磁盘
   ↓
/dev/sdb /dev/sdc
   ↓
Linux MD RAID
   ↓
/dev/md0
   ↓
LVM（可选）
   ↓
XFS / ext4
   ↓
/data
```

## 四、最常用的管理命令

先查看 RAID 总体状态：

```bash
cat /proc/mdstat
```

这是 Linux RAID 排障时最常用的第一条命令。([红帽文档][3])

查看某个 RAID 的详细信息：

```bash
mdadm --detail /dev/md0
```

查看磁盘是否包含 RAID 信息：

```bash
mdadm --examine /dev/sdb
```

创建一个简单的 RAID 1：

```bash
mdadm --create /dev/md0 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb /dev/sdc
```

创建完成后的结构：

```text
/dev/sdb ─┐
          ├── RAID 1 ──→ /dev/md0
/dev/sdc ─┘
```

然后创建文件系统：

```bash
mkfs.xfs /dev/md0
```

最后挂载：

```bash
mount /dev/md0 /data
```

Linux 发行版通常使用 `mdadm` 创建和管理软件 RAID。([红帽文档][4])

## 五、RAID 与备份不是一回事

这是 RAID 最重要的基础概念：

```text
RAID ≠ 备份
```

RAID 主要防止的是：

```text
硬盘故障
```

但它不能防止：

```text
误删除文件
病毒或勒索软件
程序错误覆盖数据
整个服务器损坏
机房灾害
```

例如，你执行：

```bash
rm -rf /data/*
```

RAID 会非常“可靠”地把删除操作同步到所有成员磁盘。

所以正确思路应该是：

```text
RAID → 保证磁盘故障时业务尽量继续运行

备份 → 保证数据能够恢复
```

## 六、初学者建议先掌握这条主线

```text
RAID 是什么
    ↓
RAID 0 / 1 / 5 / 6 / 10
    ↓
mdadm
    ↓
/proc/mdstat
    ↓
磁盘故障与降级
    ↓
更换磁盘与重建
    ↓
RAID + LVM + 文件系统
```

[1]: https://docs.kernel.org/admin-guide/md.html "RAID arrays"
[2]: https://man7.org/linux/man-pages/man8/mdadm.8.html "mdadm(8) - Linux manual page"
[3]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/5/html/deployment_guide/s1-raid-manage "6.3. Managing Software RAID | Deployment Guide"
[4]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_storage_devices/managing-raid_managing-storage-devices "Chapter 18. Managing RAID"
