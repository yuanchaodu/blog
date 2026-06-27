---
title: Linux lvm 基础
section: IT
category: Linux
---

# Linux lvm 基础

<img src="images/Linux.svg" width="300">

Linux 的 **LVM（Logical Volume Manager，逻辑卷管理）** 是一种存储管理技术，它位于**磁盘分区**和**文件系统**之间，为 Linux 提供更加灵活的磁盘管理能力。

如果把传统磁盘分区比作固定大小的房间，那么 LVM 就像可以随时拆墙、扩建、合并房间的装修系统。

---

# 一、为什么需要 LVM

传统分区存在几个问题：

* 分区大小创建后不容易调整。
* 多块磁盘不能方便地组合成一个大分区。
* 磁盘空间利用率不高。
* 后期扩容需要停机甚至重装系统。

例如：

```
/dev/sda1   100G  /
/dev/sda2   200G  /data
```

后来发现：

* `/` 空间只用了 20G
* `/data` 已经满了

传统分区很难直接把 `/` 的空间借给 `/data`。

而 LVM 可以在线扩容，大部分情况下无需重新分区。

---

# 二、LVM 的基本结构

LVM 有三个核心概念：

```
硬盘
 ↓
PV（Physical Volume）
 ↓
VG（Volume Group）
 ↓
LV（Logical Volume）
 ↓
文件系统（ext4、xfs等）
```

对应关系如下：

```
        +----------------------+
        |      Disk            |
        |   /dev/sdb1          |
        +----------------------+
                  │
                  ▼
        +----------------------+
        |   PV                |
        | Physical Volume     |
        +----------------------+
                  │
                  ▼
       +------------------------+
       |        VG              |
       | Volume Group           |
       +------------------------+
         │          │
         ▼          ▼
     +------+    +------+
     | LV1 |    | LV2 |
     +------+    +------+
        │           │
      ext4        xfs
```

---

# 三、三个重要概念

## 1、PV（Physical Volume）

物理卷。

就是把一块磁盘或者分区交给 LVM 管理。

例如：

```
/dev/sdb
```

初始化：

```bash
pvcreate /dev/sdb
```

查看：

```bash
pvs
```

或者

```bash
pvdisplay
```

---

## 2、VG（Volume Group）

卷组。

VG 就像一个"存储池"。

可以把很多 PV 放进去。

例如：

```
PV1 500G
PV2 500G

↓

VG = 1000G
```

创建：

```bash
vgcreate vg_data /dev/sdb
```

查看：

```bash
vgs
```

或

```bash
vgdisplay
```

如果以后增加一块磁盘：

```
/dev/sdc
```

先：

```bash
pvcreate /dev/sdc
```

再：

```bash
vgextend vg_data /dev/sdc
```

VG 容量立即增加。

---

## 3、LV（Logical Volume）

逻辑卷。

LV 就是真正拿来格式化、挂载的卷。

例如：

```
VG：1000G

↓

LV_home   300G
LV_data   500G
LV_backup 200G
```

创建：

```bash
lvcreate -L 200G -n lv_data vg_data
```

查看：

```bash
lvs
```

或者：

```bash
lvdisplay
```

---

# 四、LVM 工作流程

完整流程如下：

```
磁盘
↓

fdisk
↓

分区
↓

pvcreate
↓

vgcreate
↓

lvcreate
↓

mkfs.ext4

↓

mount
```

例如：

```
/dev/sdb

↓

/dev/sdb1

↓

PV

↓

VG

↓

LV

↓

ext4

↓

/data
```

---

# 五、常见命令

## 查看

```bash
pvs
vgs
lvs
```

详细信息：

```bash
pvdisplay
vgdisplay
lvdisplay
```

---

## 创建

创建 PV：

```bash
pvcreate /dev/sdb1
```

创建 VG：

```bash
vgcreate vg01 /dev/sdb1
```

创建 LV：

```bash
lvcreate -L 100G -n lv01 vg01
```

---

## 格式化

```bash
mkfs.ext4 /dev/vg01/lv01
```

或者：

```bash
mkfs.xfs /dev/vg01/lv01
```

---

## 挂载

```bash
mkdir /data

mount /dev/vg01/lv01 /data
```

永久挂载：

编辑

```text
/etc/fstab
```

添加：

```text
/dev/vg01/lv01   /data   ext4   defaults   0 0
```

---

# 六、LVM 最大优势——扩容

假设：

```
VG

1000G

↓

LV_data

200G
```

后来需要增加到：

```
500G
```

执行：

```bash
lvextend -L +300G /dev/vg01/lv_data
```

然后扩展文件系统。

如果是 ext4：

```bash
resize2fs /dev/vg01/lv_data
```

如果是 XFS：

```bash
xfs_growfs /data
```

注意：XFS 扩容时需要指定**挂载点**而不是设备名。

扩容后：

```
LV

200G

↓

500G
```

整个过程通常可以在线完成。

---

# 七、缩容（谨慎）

LVM 可以缩容，但风险较高。

对于 ext4，一般流程：

```bash
umount

↓

e2fsck

↓

resize2fs

↓

lvreduce
```

例如：

```bash
umount /data

e2fsck -f /dev/vg01/lv01

resize2fs /dev/vg01/lv01 100G

lvreduce -L 100G /dev/vg01/lv01
```

> **注意：** XFS 文件系统**不支持缩容**，只能扩容。因此在规划需要未来缩小容量的场景时，应慎重选择文件系统。

---

# 八、增加新硬盘

例如增加：

```
/dev/sdc
```

步骤：

```bash
pvcreate /dev/sdc

vgextend vg01 /dev/sdc

lvextend -l +100%FREE /dev/vg01/lv01

resize2fs ...
```

最终：

```
sdb 500G
sdc 500G

↓

VG

1000G
```

---

# 九、删除 LVM

顺序不能错。

```
卸载

↓

删除 LV

↓

删除 VG

↓

删除 PV
```

命令：

```bash
umount /data

lvremove /dev/vg01/lv01

vgremove vg01

pvremove /dev/sdb1
```

---

# 十、LVM Snapshot（快照）

LVM 可以创建快照。

用途：

* 数据备份
* 数据库热备
* 升级前回滚
* 测试环境

例如：

```bash
lvcreate -L 20G -s -n snap01 /dev/vg01/lv01
```

特点：

* 快照创建速度快。
* 初始几乎不占空间，随着原卷数据变化逐渐占用存储（写时复制机制）。
* 适合短期备份或升级保护，不建议长期保留。

---

# 十一、一个完整示例

假设新增一块磁盘 `/dev/sdb`，希望挂载到 `/data`。

```bash
# 1. 初始化物理卷
pvcreate /dev/sdb

# 2. 创建卷组
vgcreate vgdata /dev/sdb

# 3. 创建逻辑卷
lvcreate -L 200G -n lvdata vgdata

# 4. 格式化
mkfs.xfs /dev/vgdata/lvdata

# 5. 创建挂载点
mkdir /data

# 6. 挂载
mount /dev/vgdata/lvdata /data

# 7. 查看
df -h
```

如果以后再新增一块 `/dev/sdc`，可继续执行：

```bash
pvcreate /dev/sdc
vgextend vgdata /dev/sdc
lvextend -l +100%FREE /dev/vgdata/lvdata
xfs_growfs /data
```

无需重新分区或迁移数据。

---

# 十二、总结

LVM 的核心可以概括为一句话：

> **先把磁盘组成一个“资源池”（VG），再从资源池中按需划分逻辑卷（LV），最后在逻辑卷上创建文件系统并挂载使用。**

记住以下三个层次即可快速理解 LVM：

| 层次  | 名称      | 作用                    | 常用命令                        |
| --- | ------- | --------------------- | --------------------------- |
| 第一层 | PV（物理卷） | 将磁盘或分区交给 LVM 管理       | `pvcreate`、`pvs`            |
| 第二层 | VG（卷组）  | 将多个 PV 聚合为统一的存储池      | `vgcreate`、`vgextend`、`vgs` |
| 第三层 | LV（逻辑卷） | 从 VG 中划分可供文件系统使用的逻辑分区 | `lvcreate`、`lvextend`、`lvs` |

对于日常运维，最常用的命令包括：

* 查看：`pvs`、`vgs`、`lvs`
* 创建：`pvcreate`、`vgcreate`、`lvcreate`
* 扩容：`vgextend`、`lvextend`，然后根据文件系统执行 `resize2fs`（ext4）或 `xfs_growfs`（XFS）
* 删除：`lvremove` → `vgremove` → `pvremove`

掌握了 **PV → VG → LV** 的层级关系和**创建、查看、扩容**这几类操作，就已经具备了使用 LVM 管理 Linux 存储的基础能力。