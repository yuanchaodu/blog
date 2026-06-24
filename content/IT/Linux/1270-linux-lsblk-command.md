---
title: Linux lsblk 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnVz7
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/127'
---

# Linux lsblk 命令

`lsblk`（List Block Devices）是 Linux 中用于查看块设备信息的命令。它可以显示磁盘、分区、LVM、RAID、挂载点等信息，是日常运维中查看存储设备最常用的工具之一。

## 一、基本用法

```bash
lsblk
```

示例输出：

```text
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   500G  0 disk
├─sda1   8:1    0     1G  0 part /boot
├─sda2   8:2    0   100G  0 part /
└─sda3   8:3    0   399G  0 part /data
sdb      8:16   0     1T  0 disk
└─sdb1   8:17   0     1T  0 part /backup
```

字段说明：

| 字段         | 含义                 |
| ---------- | ------------------ |
| NAME       | 设备名称               |
| MAJ:MIN    | 主设备号和次设备号          |
| RM         | 是否可移动设备（1=是）       |
| SIZE       | 容量                 |
| RO         | 是否只读               |
| TYPE       | 类型（disk、part、lvm等） |
| MOUNTPOINT | 挂载点                |

---

## 二、常用参数

### 1. 查看文件系统信息

```bash
lsblk -f
```

输出示例：

```text
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 xfs          11111111-1111-1111-1111-111111111111 /boot
├─sda2 xfs          22222222-2222-2222-2222-222222222222 /
└─sda3 xfs          33333333-3333-3333-3333-333333333333 /data
```

可查看：

* 文件系统类型（xfs、ext4等）
* UUID
* LABEL
* 挂载点

---

### 2. 查看磁盘型号

```bash
lsblk -o NAME,SIZE,TYPE,MODEL
```

示例：

```text
NAME   SIZE TYPE MODEL
sda    500G disk Samsung SSD 870 EVO
sdb      1T disk ST1000DM010
```

---

### 3. 显示完整信息

```bash
lsblk -a
```

显示所有设备，包括空设备。

---

### 4. 以字节显示容量

```bash
lsblk -b
```

默认显示：

```text
500G
```

使用 `-b` 后：

```text
536870912000
```

---

### 5. 显示指定列

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
```

输出：

```text
NAME   SIZE FSTYPE MOUNTPOINT
sda    500G
├─sda1   1G xfs    /boot
├─sda2 100G xfs    /
└─sda3 399G xfs    /data
```

查看支持哪些列：

```bash
lsblk --help
```

或者：

```bash
lsblk -O
```

---

### 6. 查看 SCSI 设备

```bash
lsblk -S
```

示例：

```text
NAME HCTL       TYPE VENDOR   MODEL
sda  0:0:0:0    disk ATA      Samsung SSD
sdb  1:0:0:0    disk SEAGATE  ST1000DM010
```

常用于服务器排查 SAN 存储问题。

---

### 7. 查看设备 UUID

```bash
lsblk -o NAME,UUID
```

输出：

```text
NAME   UUID
sda1   11111111-1111-1111-1111-111111111111
sda2   22222222-2222-2222-2222-222222222222
```

---

## 三、运维常用场景

### 场景1：新增磁盘后确认系统是否识别

```bash
lsblk
```

例如新增加了一块 500G 磁盘：

```text
sdc      8:32   0   500G  0 disk
```

说明系统已经识别。

---

### 场景2：查看某个目录挂载在哪个磁盘

```bash
lsblk
```

查看：

```text
sda2 100G part /
sda3 399G part /data
```

可知：

* `/` 在 sda2
* `/data` 在 sda3

---

### 场景3：查看 LVM 结构

```bash
lsblk
```

示例：

```text
sdb
└─sdb1
  ├─vgdata-lvroot /
  └─vgdata-lvdata /data
```

可以直接看到：

* 物理卷（PV）
* 卷组（VG）
* 逻辑卷（LV）

之间的关系。

---

### 场景4：查看文件系统类型

```bash
lsblk -f
```

输出：

```text
NAME   FSTYPE
sda1   xfs
sda2   ext4
sdb1   swap
```

方便确认格式化情况。

---

## 四、与其他命令对比

| 命令          | 作用              |
| ----------- | --------------- |
| lsblk       | 查看块设备树状结构（推荐）   |
| fdisk -l    | 查看分区表详情         |
| blkid       | 查看 UUID、文件系统类型  |
| df -h       | 查看已挂载文件系统空间使用情况 |
| mount       | 查看挂载信息          |
| pvs/vgs/lvs | 查看 LVM 信息       |

---

## 五、最常用的几个命令

```bash
# 查看磁盘和分区
lsblk

# 查看文件系统
lsblk -f

# 查看磁盘型号
lsblk -o NAME,SIZE,MODEL

# 查看UUID
lsblk -o NAME,UUID

# 查看LVM结构
lsblk

# 查看所有属性
lsblk -O
```

对于日常 Linux 运维（尤其是 RHEL、CentOS、Rocky Linux、Ubuntu 服务器），最实用的是：

```bash
lsblk -f
```

它相当于把磁盘、分区、文件系统类型、UUID 和挂载点一次性展示出来，排查存储问题时使用频率最高。