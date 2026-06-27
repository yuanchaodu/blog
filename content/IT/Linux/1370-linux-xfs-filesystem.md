---
title: Linux xfs 文件系统
section: IT
category: Linux
---

# Linux xfs 文件系统

<img src="images/Linux.svg" width="300">

## 什么是 XFS 文件系统

**XFS** 是一种高性能的 **64 位日志（Journaling）文件系统**，最早由 Silicon Graphics（SGI）于 1994 年开发，后来被移植到 Linux，目前已经成为很多 Linux 发行版（如 Red Hat Enterprise Linux、Rocky Linux、AlmaLinux 等）的默认文件系统之一。

它特别适合：

* 大容量存储（TB～PB级）
* 大文件读写
* 数据库
* 日志服务器
* 视频存储
* 企业级服务器

例如，在 RHEL 7、8、9 中，默认安装时通常都会采用 XFS。

---

## XFS 的主要特点

### 1. 日志文件系统（Journal）

XFS 使用日志（Journal）记录元数据。

优点：

* 系统异常断电后恢复速度快
* 不需要长时间 fsck
* 文件系统一致性较好

它记录的是**元数据（Metadata）**，例如：

* inode
* 目录结构
* 空闲空间信息

默认**不会记录用户数据**。

---

### 2. 支持超大文件系统

XFS 最大支持：

| 项目   | 大小             |
| ---- | -------------- |
| 单文件  | 最大约 8 EiB（理论值） |
| 文件系统 | 最大约 8 EiB      |

实际上限制主要来自：

* Linux 内核
* 存储设备
* RAID
* 控制器

因此现在几十 TB、几百 TB 使用 XFS 非常常见。

---

### 3. 非常适合大文件

XFS 的优势主要体现在：

* 大视频
* 数据库
* ISO 镜像
* 备份文件
* 日志文件

例如：

```
10 GB
50 GB
100 GB
```

连续写入速度通常比 ext4 更好。

---

### 4. 延迟分配（Delayed Allocation）

XFS 不会立即分配磁盘块。

例如：

程序：

```
write()
```

只是先写入缓存。

真正落盘时：

```
flush
sync
```

系统会一次性分配连续空间。

优点：

* 减少碎片
* 提高顺序写性能

---

### 5. Extent（区段）管理

传统文件系统：

```
Block
```

例如：

```
文件

block1
block2
block3
block4
...
```

XFS 使用 Extent：

```
Extent
```

例如：

```
起始块：1000

长度：500 blocks
```

一个 Extent 就表示：

```
1000~1499
```

而不是记录 500 个 Block。

因此：

* inode 更小
* 元数据更少
* 查询速度更快

---

### 6. B+Tree

XFS 内部大量使用 **B+ tree**。

包括：

* inode
* Extent
* 空闲空间
* 目录

因此即使：

```
1000 万文件
```

查找效率仍然很高。

---

### 7. Allocation Group（AG）

这是 XFS 最大的特点。

一个文件系统会划分成多个：

```
AG
```

例如：

```
10TB

↓

AG0
AG1
AG2
AG3
...
```

每个 AG：

* 有自己的 inode
* 有自己的空闲空间
* 有自己的 B+Tree

多个 CPU 可以同时操作不同 AG。

因此：

多线程性能非常优秀。

---

## XFS 与 ext4 对比

| 特性         | XFS     | ext4     |
| ---------- | ------- | -------- |
| 默认日志       | 是       | 是        |
| 大文件性能      | ★★★★★   | ★★★★☆    |
| 小文件性能      | ★★★★☆   | ★★★★★    |
| 大目录        | ★★★★★   | ★★★★☆    |
| TB/PB 文件系统 | 非常好     | 较好       |
| 并发性能       | ★★★★★   | ★★★☆☆    |
| 在线扩容       | 支持      | 支持       |
| 在线缩容       | **不支持** | 不支持      |
| 离线缩容       | **不支持** | 支持（部分场景） |
| 碎片控制       | 较好      | 一般       |

---

## XFS 常用命令

### 创建文件系统

```bash
mkfs.xfs /dev/sdb1
```

指定标签：

```bash
mkfs.xfs -L data /dev/sdb1
```

---

### 查看文件系统

```bash
xfs_info /dev/sdb1
```

输出示例：

```
meta-data=/dev/sdb1
isize=512
agcount=4
agsize=...
```

---

### 挂载

```bash
mount /dev/sdb1 /data
```

查看：

```bash
df -Th
```

例如：

```
Filesystem Type Size Used Avail
/dev/sdb1 xfs 2T ...
```

---

### 在线扩容

这是 XFS 最常用的功能之一。

假设已经扩展了底层磁盘或逻辑卷（LVM）：

```bash
xfs_growfs /data
```

也可以指定设备：

```bash
xfs_growfs /dev/sdb1
```

注意：

必须已经挂载。

---

### 修复

检查：

```bash
xfs_repair /dev/sdb1
```

如果日志损坏：

```bash
xfs_repair -L /dev/sdb1
```

**注意：** `-L` 会清空日志，可能导致少量最近写入的数据丢失，应作为最后的修复手段。

---

### 查看碎片

```bash
xfs_db
```

或者：

```bash
xfs_fsr
```

用于碎片整理。

---

## XFS 与 LVM 配合使用

企业环境中，XFS 常与 Logical Volume Manager 配合使用，实现不停机扩容：

```
磁盘
   │
PV（物理卷）
   │
VG（卷组）
   │
LV（逻辑卷）
   │
XFS
```

例如：

```bash
lvextend -L +500G /dev/vgdata/lvdata
xfs_growfs /data
```

整个过程通常无需卸载文件系统。

---

## XFS 的缺点

虽然 XFS 性能优秀，但也存在一些限制：

* **不能缩容**：XFS 支持扩容，但无论在线还是离线，都不支持缩小文件系统，因此容量规划要留有余地。
* **小文件场景不一定占优**：在包含大量小文件的工作负载下，ext4 有时会表现更好。
* **修复耗时**：极大容量文件系统发生损坏时，`xfs_repair` 可能耗时较长。
* **日志恢复能力有限**：日志主要保护元数据，不能替代备份，不能保证用户数据在异常断电时绝对不丢失。

---

## 企业应用场景

XFS 特别适合以下场景：

* 数据库服务器（如 Oracle Database、MySQL、PostgreSQL）
* 大数据平台
* NAS 存储
* 日志服务器
* 视频监控存储
* 虚拟化平台（如 VMware ESXi、KVM）
* 大型文件服务器

## 总结

XFS 的核心优势可以概括为：

* **高性能**：适合大文件、高吞吐量和高并发场景。
* **高扩展性**：支持超大容量文件系统和在线扩容。
* **高可靠性**：采用日志机制，异常恢复速度快。
* **高并发能力**：通过 Allocation Group（AG）设计，充分利用多核 CPU。

对于企业生产环境，尤其是数据库、日志、备份、虚拟化和大容量存储场景，XFS 通常是 Linux 平台上最成熟、最稳定的文件系统之一。如果需要频繁缩容文件系统或主要存放大量小文件，则可以结合业务特点考虑 ext4 等其他文件系统。