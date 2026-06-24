---
title: Linux 磁盘介绍
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnVzY
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/126'
---

# Linux 磁盘介

<img src="images/Linux.svg" width="300">

Linux 中的磁盘管理是系统管理的重要基础。无论是服务器、虚拟机还是个人电脑，数据最终都存储在磁盘设备中。理解 Linux 的磁盘结构和管理方式，有助于做好系统部署、容量规划、性能优化和故障排查。

# 一、Linux 磁盘体系结构

Linux 将几乎所有硬件都抽象成文件，磁盘设备通常位于 `/dev` 目录下。

常见磁盘名称：

| 设备名称           | 说明                   |
| -------------- | -------------------- |
| `/dev/sda`     | 第一块 SATA/SAS/SCSI 磁盘 |
| `/dev/sdb`     | 第二块磁盘                |
| `/dev/nvme0n1` | 第一块 NVMe SSD         |
| `/dev/vda`     | KVM 虚拟机磁盘            |
| `/dev/xvda`    | Xen 虚拟机磁盘            |

例如：

```bash
/dev/sda
```

表示第一块磁盘。

```bash
/dev/sda1
```

表示第一块磁盘的第一个分区。

NVMe 磁盘命名方式不同：

```bash
/dev/nvme0n1p1
```

表示第一块 NVMe 磁盘的第一个分区。

---

# 二、磁盘、分区和文件系统

Linux 磁盘管理一般分为三层：

```text
物理磁盘
    ↓
磁盘分区
    ↓
文件系统
    ↓
挂载目录
```

例如：

```text
磁盘：/dev/sda
    ↓
分区：/dev/sda1
    ↓
EXT4文件系统
    ↓
挂载到 /data
```

用户访问：

```bash
/data/test.txt
```

实际上最终存储在 `/dev/sda1` 中。

可以理解为：

* 磁盘 = 仓库
* 分区 = 仓库中的区域
* 文件系统 = 货架管理规则
* 挂载点 = 仓库入口

---

# 三、查看磁盘信息

## 1. lsblk

最常用命令：

```bash
lsblk
```

示例：

```text
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda       8:0    0 500G  0 disk
├─sda1    8:1    0   1G  0 part /boot
└─sda2    8:2    0 499G  0 part /
```

说明：

* disk：磁盘
* part：分区
* MOUNTPOINT：挂载点

---

## 2. fdisk

查看分区信息：

```bash
fdisk -l
```

输出示例：

```text
Disk /dev/sda: 500 GiB
Disklabel type: gpt
```

可查看：

* 磁盘容量
* 分区数量
* 分区类型
* GPT/MBR格式

---

## 3. df

查看文件系统使用情况：

```bash
df -h
```

示例：

```text
Filesystem      Size Used Avail Use%
/dev/sda2       499G 100G 399G 20%
```

参数：

```bash
-h
```

表示人类可读格式（GB、TB）。

---

## 4. blkid

查看文件系统类型和 UUID：

```bash
blkid
```

输出：

```text
/dev/sda1: UUID="xxxx" TYPE="xfs"
/dev/sda2: UUID="yyyy" TYPE="ext4"
```

---

# 四、分区表类型

Linux 主要支持两种分区表。

## 1. MBR（Master Boot Record）

特点：

* 最大支持 2TB
* 最多 4 个主分区
* 历史悠久

结构：

```text
MBR
├─ 主分区1
├─ 主分区2
├─ 主分区3
└─ 扩展分区
      ├─逻辑分区
      ├─逻辑分区
```

---

## 2. GPT（GUID Partition Table）

目前主流方案。

特点：

* 支持超过 2TB
* 支持 128 个以上分区
* 配合 UEFI 启动

查看：

```bash
parted -l
```

输出：

```text
Partition Table: gpt
```

---

# 五、常见文件系统

文件系统决定数据如何组织和存储。

## 1. EXT4

Linux 默认文件系统之一。

特点：

* 稳定可靠
* 管理简单
* 应用广泛

创建：

```bash
mkfs.ext4 /dev/sdb1
```

---

## 2. XFS

企业级服务器常用。

特点：

* 大文件性能好
* 支持在线扩容
* Red Hat 系列默认

创建：

```bash
mkfs.xfs /dev/sdb1
```

---

## 3. Btrfs

新一代文件系统。

特点：

* 快照
* 校验
* 压缩
* 子卷管理

类似：

* VMware Snapshot
* 存储快照

---

## 4. FAT32 / NTFS

主要用于与 Windows 共享数据。

```bash
mkfs.vfat
```

```bash
mkfs.ntfs
```

---

# 六、挂载（Mount）

Linux 不使用 Windows 的盘符（C盘、D盘）。

所有文件统一挂载到目录树。

例如：

```text
/
├── home
├── var
├── data
└── backup
```

挂载：

```bash
mount /dev/sdb1 /data
```

查看：

```bash
mount
```

或：

```bash
df -h
```

---

# 七、自动挂载

系统启动后自动挂载配置文件：

```bash
/etc/fstab
```

示例：

```text
UUID=xxxx /data ext4 defaults 0 0
```

验证配置：

```bash
mount -a
```

无报错说明配置正确。

---

# 八、LVM（逻辑卷管理）

企业服务器广泛使用。

传统方式：

```text
磁盘
 ↓
分区
 ↓
文件系统
```

LVM方式：

```text
磁盘
 ↓
PV
 ↓
VG
 ↓
LV
 ↓
文件系统
```

## 组成

### PV（Physical Volume）

物理卷

```bash
pvcreate /dev/sdb
```

---

### VG（Volume Group）

卷组

```bash
vgcreate vgdata /dev/sdb
```

---

### LV（Logical Volume）

逻辑卷

```bash
lvcreate -L 100G -n lvdata vgdata
```

---

优点：

* 动态扩容
* 多磁盘合并
* 容量管理灵活

例如：

```text
磁盘1：500G
磁盘2：500G

合并后：

VG = 1TB
```

使用时就像一个大磁盘。

---

# 九、RAID 简介

RAID 用于提高性能或可靠性。

## RAID0

```text
磁盘A + 磁盘B
```

特点：

* 容量叠加
* 性能提升
* 无冗余

任何一块盘损坏：

```text
数据全部丢失
```

---

## RAID1

```text
磁盘A ↔ 磁盘B
```

特点：

* 镜像备份
* 容错能力强

缺点：

```text
2块盘只能用1块容量
```

---

## RAID5

```text
至少3块盘
```

特点：

* 数据+校验分布存储
* 允许损坏1块盘

企业最常见。

---

## RAID10

```text
RAID1 + RAID0
```

特点：

* 高性能
* 高可靠

数据库服务器常用。

---

# 十、磁盘性能指标

## IOPS

每秒输入输出次数。

适合：

* 数据库
* 虚拟化平台

例如：

```text
SSD：50000+
机械盘：100~200
```

---

## 吞吐量（Throughput）

单位时间传输数据量。

例如：

```text
500MB/s
```

适用于：

* 文件服务器
* 视频存储

---

## 延迟（Latency）

一次IO响应时间。

例如：

```text
1ms
```

越小越好。

---

# 十一、常用磁盘管理命令速查

| 功能     | 命令                 |
| ------ | ------------------ |
| 查看磁盘   | `lsblk`            |
| 查看分区   | `fdisk -l`         |
| 查看容量   | `df -h`            |
| 查看UUID | `blkid`            |
| 查看挂载   | `mount`            |
| 创建EXT4 | `mkfs.ext4`        |
| 创建XFS  | `mkfs.xfs`         |
| 挂载磁盘   | `mount`            |
| 卸载磁盘   | `umount`           |
| 查看LVM  | `pvs/vgs/lvs`      |
| 查看RAID | `cat /proc/mdstat` |

# 十二、企业生产环境中的典型规划

以 Linux 服务器为例：

```text
系统盘（SSD）
├── /boot      1G
├── /          100G
├── swap       16G
└── /var       100G

数据盘（RAID）
├── /data
├── /backup
└── /archive
```

推荐实践：

1. 使用 GPT 分区表。
2. 使用 XFS 或 EXT4 文件系统。
3. 数据盘采用 LVM 管理。
4. 重要数据使用 RAID1、RAID5 或 RAID10。
5. 通过 UUID 配置 `/etc/fstab`。
6. 定期监控磁盘空间（`df -h`）和磁盘健康状态（SMART）。

如果从企业运维和数字化工厂建设角度来看，下一步最值得深入学习的是 **Linux 磁盘扩容（LVM 扩容）、XFS/EXT4 文件系统管理、RAID 管理以及存储性能分析（iostat、iotop、sar）**，这些是生产环境中最常见的磁盘运维工作。
