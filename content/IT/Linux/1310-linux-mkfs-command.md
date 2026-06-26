---
title: Linux mkfs 命令
section: IT
category: Linux
---

# Linux mkfs 命令

<img src="images/Linux.svg" width="300">

`mkfs`（**make filesystem**）是 Linux 中用于**在磁盘分区或块设备上创建文件系统**的命令。通常是在新磁盘、U 盘或新分区完成分区之后使用。

## 一、命令格式

```bash
mkfs [选项] [-t 文件系统类型] 设备名
```

或者直接使用对应的文件系统命令：

```bash
mkfs.ext4
mkfs.xfs
mkfs.vfat
mkfs.btrfs
```

例如：

```bash
mkfs.ext4 /dev/sdb1
```

这实际上等价于：

```bash
mkfs -t ext4 /dev/sdb1
```

---

## 二、常见文件系统类型

| 文件系统  | 命令           | 用途           |
| ----- | ------------ | ------------ |
| ext4  | `mkfs.ext4`  | Linux 默认、最常用 |
| xfs   | `mkfs.xfs`   | 大容量存储、高性能服务器 |
| ext3  | `mkfs.ext3`  | 较老系统         |
| ext2  | `mkfs.ext2`  | 无日志文件系统      |
| FAT32 | `mkfs.vfat`  | Windows、U盘兼容 |
| exFAT | `mkfs.exfat` | 大容量移动设备      |
| NTFS  | `mkfs.ntfs`  | Windows 文件系统 |
| Btrfs | `mkfs.btrfs` | 快照、压缩、校验     |

---

## 三、常用示例

### 1. 创建 ext4 文件系统

```bash
mkfs.ext4 /dev/sdb1
```

输出类似：

```
mke2fs 1.46.5
Creating filesystem with 26214400 4k blocks...
Writing inode tables...
Creating journal...
Writing superblocks...
done
```

---

### 2. 使用 mkfs 指定类型

```bash
mkfs -t ext4 /dev/sdb1
```

---

### 3. 创建 XFS

```bash
mkfs.xfs /dev/sdb1
```

或者

```bash
mkfs -t xfs /dev/sdb1
```

---

### 4. 创建 FAT32

```bash
mkfs.vfat -F 32 /dev/sdb1
```

参数说明：

* `-F 32`
  指定 FAT32。

---

### 5. 创建 NTFS

```bash
mkfs.ntfs /dev/sdb1
```

---

## 四、常用参数

| 参数   | 作用                |
| ---- | ----------------- |
| `-t` | 指定文件系统类型          |
| `-V` | 显示详细执行过程          |
| `-L` | 设置卷标（部分文件系统）      |
| `-F` | 强制执行（不同文件系统含义不同）  |
| `-b` | 指定块大小             |
| `-m` | 设置保留空间百分比（ext 系列） |

例如：

设置卷标：

```bash
mkfs.ext4 -L DATA /dev/sdb1
```

查看：

```bash
lsblk -f
```

输出：

```
NAME   FSTYPE LABEL UUID
sdb1   ext4   DATA  xxxxx
```

---

### 设置块大小

例如：

```bash
mkfs.ext4 -b 4096 /dev/sdb1
```

表示：

```
Block Size = 4KB
```

---

### 设置 root 保留空间

默认 ext4 会保留 **5%** 空间给 root 用户。

例如：

```bash
mkfs.ext4 -m 1 /dev/sdb1
```

表示：

```
保留 1%
```

---

## 五、使用流程

假设新增一块磁盘 `/dev/sdb`。

### 第一步：查看磁盘

```bash
lsblk
```

输出：

```
sda
├─sda1
└─sda2

sdb
```

---

### 第二步：分区

使用：

```bash
fdisk /dev/sdb
```

或者：

```bash
parted /dev/sdb
```

建立：

```
/dev/sdb1
```

---

### 第三步：格式化

```bash
mkfs.ext4 /dev/sdb1
```

---

### 第四步：创建挂载点

```bash
mkdir /data
```

---

### 第五步：挂载

```bash
mount /dev/sdb1 /data
```

查看：

```bash
df -h
```

输出：

```
Filesystem      Size Used Avail Mounted on
/dev/sdb1       100G   1G   94G /data
```

---

## 六、查看文件系统

查看文件系统类型：

```bash
lsblk -f
```

或者：

```bash
blkid
```

例如：

```
/dev/sdb1: UUID="xxxx"
TYPE="ext4"
```

---

## 七、查看 mkfs 支持哪些文件系统

```bash
mkfs.
```

然后按 **Tab** 键自动补全，通常可以看到：

```
mkfs.bfs
mkfs.btrfs
mkfs.cramfs
mkfs.ext2
mkfs.ext3
mkfs.ext4
mkfs.fat
mkfs.minix
mkfs.vfat
mkfs.xfs
```

也可以查看帮助：

```bash
mkfs --help
```

---

## 八、注意事项

1. **`mkfs` 会清空目标设备上的所有数据**，操作前请确认设备名称无误。

2. **不要对正在挂载使用中的分区执行 `mkfs`**。可先用以下命令确认挂载情况：

   ```bash
   mount | grep sdb1
   ```

   或：

   ```bash
   lsblk
   ```

3. 建议优先使用对应文件系统的专用命令（如 `mkfs.ext4`、`mkfs.xfs`），因为它们支持更多针对该文件系统的参数和优化选项。

---

## 九、`mkfs` 与相关命令的关系

| 命令                 | 功能              | 是否删除数据               |
| ------------------ | --------------- | -------------------- |
| `fdisk` / `parted` | 创建或修改磁盘分区       | 一般不直接删除数据（但调整分区存在风险） |
| `mkfs`             | 在分区上创建文件系统（格式化） | **是**                |
| `mount`            | 挂载文件系统          | 否                    |
| `umount`           | 卸载文件系统          | 否                    |
| `fsck`             | 检查并修复文件系统       | 否（可能修复损坏）            |
| `blkid`            | 查看文件系统类型和 UUID  | 否                    |
| `lsblk -f`         | 查看块设备、文件系统和挂载信息 | 否                    |

**典型流程**为：**分区（`fdisk`/`parted`）→ 格式化（`mkfs`）→ 挂载（`mount`）→ 配置自动挂载（编辑 `/etc/fstab`）**。这是 Linux 中新增磁盘并投入使用的标准步骤。