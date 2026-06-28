---
title: Linux lvm 高级管理
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4Ana2x
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/139'
---

# Linux lvm 高级管理

<img src="images/Linux.svg" width="300">

Linux LVM（Logical Volume Manager，逻辑卷管理）高级管理主要涉及逻辑卷的在线扩容、缩容、迁移、快照、精简配置（Thin Provisioning）、缓存、RAID、故障恢复等内容。如果已经掌握了 PV、VG、LV 的基本概念，那么下面这些才是真正生产环境中经常使用的高级功能。

---

# 一、LVM结构回顾

```
磁盘
 ├── /dev/sdb
 ├── /dev/sdc
      │
      ▼
PV（Physical Volume）
      │
      ▼
VG（Volume Group）
      │
      ├──── LV1
      ├──── LV2
      └──── LV3
```

查看关系

```bash
pvs
vgs
lvs -a
```

或者

```bash
pvdisplay
vgdisplay
lvdisplay
```

---

# 二、在线扩容（生产最常见）

假设：

```
VG：vgdata

LV：lvdata

文件系统：xfs
```

## 第一步：增加磁盘

例如新增

```
/dev/sdc
```

初始化

```bash
pvcreate /dev/sdc
```

查看

```bash
pvs
```

---

## 第二步：加入VG

```bash
vgextend vgdata /dev/sdc
```

查看

```bash
vgs
```

容量已经增加。

---

## 第三步：扩展LV

增加100G

```bash
lvextend -L +100G /dev/vgdata/lvdata
```

或者

```bash
lvextend -l +100%FREE /dev/vgdata/lvdata
```

表示使用VG全部剩余空间。

---

## 第四步：扩展文件系统

### XFS

```bash
xfs_growfs /data
```

注意：

XFS只能扩容。

---

### EXT4

```bash
resize2fs /dev/vgdata/lvdata
```

---

也可以一步完成

```bash
lvextend -r -L +100G /dev/vgdata/lvdata
```

-r 自动执行 resize。

---

# 三、缩容（风险最高）

> **XFS 不支持缩容。**

只有 EXT4 可以缩容。

流程：

卸载

```bash
umount /data
```

检查

```bash
e2fsck -f /dev/vgdata/lvdata
```

缩小文件系统

```bash
resize2fs /dev/vgdata/lvdata 100G
```

缩小LV

```bash
lvreduce -L 100G /dev/vgdata/lvdata
```

重新挂载

```bash
mount /data
```

---

生产环境建议：

**先缩文件系统，再缩LV。**

千万不要反过来。

---

# 四、迁移数据（pvmove）

例如

```
/dev/sdb
```

需要更换。

新增

```
/dev/sdd
```

初始化

```bash
pvcreate /dev/sdd
vgextend vgdata /dev/sdd
```

迁移

```bash
pvmove /dev/sdb /dev/sdd
```

完成以后

```bash
vgreduce vgdata /dev/sdb
```

最后

```bash
pvremove /dev/sdb
```

整个过程：

```
旧盘
↓↓↓↓↓↓↓↓

数据

↓↓↓↓↓↓↓↓

新盘

业务无需停止
```

这是LVM最大的优势之一。

---

# 五、LVM快照（Snapshot）

例如

```
lvdata
```

创建快照

```bash
lvcreate -L 20G -s -n lv_snap /dev/vgdata/lvdata
```

参数：

```
-s
```

表示Snapshot。

查看

```bash
lvs
```

会看到

```
Origin

Snapshot
```

---

挂载快照

```bash
mount /dev/vgdata/lv_snap /mnt
```

即可读取历史数据。

---

删除

```bash
lvremove /dev/vgdata/lv_snap
```

---

快照特点

优点：

* 秒级创建
* 数据一致性
* 适合备份

缺点：

* Snapshot越久性能越差
* 修改越多占用越大
* 不建议长期存在

---

# 六、Thin Provisioning（精简配置）

普通LVM

```
VG 1TB

LV只能分1TB
```

Thin LVM

```
VG 1TB

可以创建

10TB

逻辑卷
```

只有真正写数据才占空间。

创建Pool

```bash
lvcreate -L 500G -T vgdata/thinpool
```

创建Thin LV

```bash
lvcreate -V 2T -T vgdata/thinpool -n thinlv
```

说明：

```
实际500G

逻辑2T
```

这是云平台（如VMware、OpenStack、KVM）中常用的技术。

查看

```bash
lvs -a
```

会看到

```
Pool

Thin
```

---

# 七、LVM Cache

利用SSD加速HDD。

例如

```
HDD

/dev/sdb
```

SSD

```
/dev/nvme0n1
```

建立Cache Pool

```bash
lvcreate --type cache-pool
```

绑定

```bash
lvconvert --type cache
```

这样：

```
热点数据

SSD

↓

冷数据

HDD
```

数据库性能可以明显提升。

---

# 八、LVM RAID

LVM支持RAID。

例如

RAID1

```bash
lvcreate --type raid1
```

RAID5

```bash
lvcreate --type raid5
```

RAID6

```bash
lvcreate --type raid6
```

例如

```bash
lvcreate \
-L 500G \
--type raid1 \
-m1 \
-n lvdata \
vgdata
```

查看

```bash
lvs -a
```

会看到

```
rimage

rmeta
```

---

# 九、Striping（条带）

类似RAID0。

创建

```bash
lvcreate -i2 -I64 -L500G -n lvdata vgdata
```

参数：

```
-i2
```

两块盘

```
-I64
```

Stripe大小64KB。

适合：

* 大文件
* 视频
* HPC
* 数据仓库

---

# 十、镜像（Mirror）

老版本

```bash
lvcreate -m1
```

现在建议使用

```
RAID1
```

代替Mirror。

---

# 十一、磁盘替换

假设

```
/dev/sdb

坏了
```

新增

```
/dev/sdd
```

```bash
pvcreate /dev/sdd
vgextend vgdata /dev/sdd

pvmove /dev/sdb /dev/sdd

vgreduce vgdata /dev/sdb

pvremove /dev/sdb
```

整个业务不中断。

---

# 十二、VG拆分

把VG拆出去。

```bash
vgsplit
```

例如

```
VG

vgdata

↓

vgdata

+

vgbackup
```

---

# 十三、VG合并

```bash
vgmerge
```

例如

```
vg1

+

vg2

↓

vgall
```

---

# 十四、导入导出VG

服务器迁移

导出

```bash
vgexport vgdata
```

另一台机器

```bash
vgimport vgdata
```

即可识别。

---

# 十五、LVM恢复

扫描

```bash
pvscan
vgscan
lvscan
```

恢复VG

```bash
vgcfgrestore vgdata
```

查看备份

```bash
ls /etc/lvm/archive/
```

查看自动备份

```bash
ls /etc/lvm/backup/
```

LVM默认保存VG元数据。

---

# 十六、查看映射关系

查看详细信息

```bash
dmsetup ls
```

查看

```bash
lsblk
```

例如

```
sdb
 └──vgdata-lvdata
```

或者

```bash
lvs -a -o +devices
```

输出示例：

```
LV      VG      Devices
lvdata  vgdata  /dev/sdb(0)
```

如果是跨多个PV：

```
lvdata vgdata /dev/sdb(0),/dev/sdc(10000)
```

---

# 十七、常用维护命令

| 操作      | 命令                |
| ------- | ----------------- |
| 查看PV    | `pvs`             |
| 查看VG    | `vgs`             |
| 查看LV    | `lvs -a`          |
| 查看底层设备  | `lvs -o +devices` |
| 查看映射    | `lsblk`           |
| 创建PV    | `pvcreate`        |
| 扩展VG    | `vgextend`        |
| 扩展LV    | `lvextend -r`     |
| 缩减LV    | `lvreduce`        |
| 迁移数据    | `pvmove`          |
| 删除PV    | `pvremove`        |
| 删除LV    | `lvremove`        |
| 删除VG    | `vgremove`        |
| 创建快照    | `lvcreate -s`     |
| 恢复VG    | `vgcfgrestore`    |
| 查看LVM配置 | `lvmconfig`       |
| 查看设备映射  | `dmsetup ls`      |

---

# 十八、生产环境最佳实践

1. **优先使用 XFS 文件系统**：在现代 Linux 发行版（如 RHEL、CentOS、Rocky、AlmaLinux）中，XFS 是默认选择，支持在线扩容，但不支持缩容。如果未来可能需要缩容，可考虑 EXT4。
2. **扩容优先使用 `lvextend -r`**：该命令可同时扩展逻辑卷和文件系统，减少人工操作步骤和出错风险。
3. **避免频繁使用快照**：传统 LVM 快照适合短期备份或一致性检查，长时间保留会影响性能。对于频繁快照需求，可考虑 Thin Snapshot。
4. **定期备份 LVM 元数据**：虽然 `/etc/lvm/backup/` 和 `/etc/lvm/archive/` 会自动保存配置，但建议将其纳入系统备份策略，以便在 VG 元数据损坏时快速恢复。
5. **使用 `pvmove` 实现在线数据迁移**：更换磁盘、存储扩容或设备维护时，可在业务不中断的情况下完成数据迁移。
6. **监控 VG 剩余空间**：特别是在使用 Thin Provisioning 时，应重点关注数据池（Data%）和元数据池（Meta%）使用率，避免超分配导致写入失败。
7. **在高可用场景优先使用 LVM RAID**：相比早期的 Mirror 实现，LVM RAID 功能更完善，也更符合当前维护方向。
8. **对于数据库或关键业务，在执行扩容、迁移、快照等操作前，先做好数据备份和恢复验证**，确保在极端情况下能够快速回退。
