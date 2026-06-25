---
title: Linux fdisk 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnXaw
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/129'
---

# Linux fdisk 命令

<img src="images/Linux.svg" width="300">

`fdisk` 是 Linux 中最常用的**磁盘分区管理工具**之一，主要用于创建、删除、修改磁盘分区。它适用于 **MBR** 和 **GPT** 分区表（较新版本的 `fdisk` 已支持 GPT），但更推荐用于交互式管理分区。

---

# 一、fdisk 的基本语法

```bash
fdisk [选项] <磁盘设备>
```

例如：

```bash
sudo fdisk /dev/sdb
```

这里操作的是整个磁盘，而不是某个分区。

常见磁盘名称：

```
/dev/sda      第一块SATA/SCSI硬盘
/dev/sdb      第二块硬盘
/dev/nvme0n1  NVMe SSD
/dev/vda      KVM虚拟机磁盘
```

---

# 二、查看磁盘分区

## 方法一：fdisk -l

查看所有磁盘

```bash
sudo fdisk -l
```

输出示例：

```text
Disk /dev/sda: 500 GiB

Device      Start       End   Sectors   Size Type
/dev/sda1    2048   1050623   1048576   512M EFI System
/dev/sda2 1050624 976773119 975722496 465.3G Linux filesystem
```

查看指定磁盘

```bash
sudo fdisk -l /dev/sdb
```

---

# 三、进入交互模式

```bash
sudo fdisk /dev/sdb
```

进入后会看到：

```text
Command (m for help):
```

输入：

```text
m
```

即可查看帮助。

---

# 四、常用命令

| 命令 | 功能              |
| -- | --------------- |
| m  | 查看帮助            |
| p  | 显示分区表           |
| n  | 新建分区            |
| d  | 删除分区            |
| t  | 修改分区类型          |
| a  | 设置启动标志（Boot）    |
| w  | 保存并退出           |
| q  | 不保存退出           |
| g  | 创建 GPT 分区表      |
| o  | 创建 DOS(MBR) 分区表 |
| l  | 查看支持的分区类型       |

---

# 五、创建新分区

例如给 `/dev/sdb` 创建一个分区。

进入：

```bash
sudo fdisk /dev/sdb
```

执行：

```
n
```

系统询问：

```
Partition type
   p   primary
   e   extended
```

对于 GPT：

```
Partition number (1-128)
```

直接回车：

```
First sector
```

回车：

```
Last sector
```

可以输入：

```
+20G
```

表示创建 20GB 分区。

最后输入：

```
w
```

保存。

---

# 六、删除分区

进入：

```bash
sudo fdisk /dev/sdb
```

输入：

```
d
```

如果有多个分区：

```
Partition number (1,2,3...)
```

输入要删除的编号。

最后：

```
w
```

保存。

> **注意：** 删除分区会导致该分区中的数据无法正常访问，请提前备份重要数据。

---

# 七、修改分区类型

输入：

```
t
```

例如：

```
Hex code or alias
```

可以输入：

```
83
```

表示 Linux filesystem。

查看支持类型：

```
L
```

常见类型：

| 编号 | 类型         |
| -- | ---------- |
| 83 | Linux      |
| 82 | Linux Swap |
| EF | EFI System |
| 07 | NTFS       |
| 0C | FAT32(LBA) |

GPT 下显示的是 GUID 类型，名称更直观。

---

# 八、查看当前分区

输入：

```
p
```

输出例如：

```text
Device      Start      End  Sectors Size Type
/dev/sdb1    2048 41945087 41943040  20G Linux filesystem
```

---

# 九、保存退出

```
w
```

保存所有修改。

退出不保存：

```
q
```

---

# 十、让内核重新读取分区表

修改完成后，有时需要通知内核重新读取分区信息。

方法一：

```bash
sudo partprobe
```

方法二：

```bash
sudo partx -u /dev/sdb
```

或者重启系统。

---

# 十一、格式化新分区

例如：

```bash
sudo mkfs.ext4 /dev/sdb1
```

或者：

```bash
sudo mkfs.xfs /dev/sdb1
```

查看：

```bash
lsblk -f
```

输出示例：

```text
NAME   FSTYPE LABEL UUID
sdb
└─sdb1 ext4
```

---

# 十二、挂载分区

创建挂载目录：

```bash
sudo mkdir /data
```

挂载：

```bash
sudo mount /dev/sdb1 /data
```

查看：

```bash
df -h
```

---

# 十三、查看磁盘结构

相比 `fdisk -l`，`lsblk` 更直观：

```bash
lsblk
```

示例：

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 500G  0 disk
├─sda1   8:1    0 512M  0 part /boot
└─sda2   8:2    0 499G  0 part /
sdb      8:16   0 100G  0 disk
└─sdb1   8:17   0 100G  0 part
```

树形结构更容易理解磁盘与分区的关系。

---

# 十四、注意事项

1. **操作对象是磁盘设备**：例如 `fdisk /dev/sdb`，不要直接对分区（如 `/dev/sdb1`）运行 `fdisk`。
2. **修改分区表前备份重要数据**：错误操作可能导致数据丢失。
3. **保存前可多次检查**：在交互模式下使用 `p` 查看当前分区布局，确认无误后再执行 `w`。
4. **新分区需格式化并挂载**：仅创建分区后还不能直接使用，需要使用 `mkfs` 创建文件系统并挂载。
5. **大容量磁盘建议使用 GPT**：对于容量超过 **2 TB** 的磁盘，推荐使用 GPT 分区表；MBR 存在约 **2 TB** 容量限制且最多支持 4 个主分区。

---

## 常用命令速查

| 操作          | 命令                    |
| ----------- | --------------------- |
| 查看所有磁盘和分区   | `sudo fdisk -l`       |
| 管理磁盘        | `sudo fdisk /dev/sdb` |
| 查看当前分区      | `p`                   |
| 新建分区        | `n`                   |
| 删除分区        | `d`                   |
| 修改分区类型      | `t`                   |
| 查看帮助        | `m`                   |
| 保存退出        | `w`                   |
| 不保存退出       | `q`                   |
| 创建 GPT 分区表  | `g`                   |
| 创建 MBR 分区表  | `o`                   |
| 通知内核重新读取分区表 | `sudo partprobe`      |
| 查看块设备和挂载情况  | `lsblk -f`            |

对于日常运维，如果只是查看磁盘和分区信息，通常推荐优先使用 `lsblk`、`blkid` 和 `df -h`；只有在需要创建、删除或调整分区时，再使用 `fdisk` 或功能更丰富的 `parted`。
