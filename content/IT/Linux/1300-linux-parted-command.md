---
title: Linux parted 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnYss
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/130'
---

# Linux parted 命令

<img src="images/Linux.svg" width="300">

`parted` 是 Linux 下用于**磁盘分区管理**的工具，相比传统的 `fdisk`，它支持 **GPT 分区表**、大于 2TB 的磁盘、在线调整部分分区等功能，因此目前大多数 Linux 发行版都推荐使用 `parted`。

---

# 一、parted 的作用

主要用于：

* 查看磁盘信息
* 创建分区
* 删除分区
* 修改分区大小（部分文件系统）
* 创建 GPT/MBR 分区表
* 设置分区标志（boot、esp 等）

支持：

* GPT（推荐）
* MBR（msdos）
* Apple Partition Map
* BSD Label 等

---

# 二、基本语法

```bash
parted [选项] 设备
```

例如

```bash
parted /dev/sdb
```

进入交互模式。

也可以直接执行命令：

```bash
parted /dev/sdb print
```

---

# 三、查看磁盘

查看所有磁盘

```bash
lsblk
```

或

```bash
fdisk -l
```

然后查看某块磁盘：

```bash
parted /dev/sdb print
```

例如输出：

```
Model: VMware Virtual Disk
Disk /dev/sdb: 100GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End      Size     File system  Name   Flags
 1      1049kB  538MB    537MB    fat32              boot, esp
 2      538MB   100GB    99.5GB   ext4
```

说明：

| 字段          | 说明   |
| ----------- | ---- |
| Start       | 起始位置 |
| End         | 结束位置 |
| Size        | 大小   |
| File system | 文件系统 |
| Flags       | 启动标志 |

---

# 四、创建 GPT 分区表

⚠️ 会删除磁盘所有分区。

```bash
parted /dev/sdb
```

执行：

```
mklabel gpt
```

或者直接：

```bash
parted /dev/sdb mklabel gpt
```

创建 MBR：

```bash
parted /dev/sdb mklabel msdos
```

---

# 五、创建分区

例如：

创建一个 20GB 分区

```bash
parted /dev/sdb
```

执行：

```
mkpart primary ext4 1MiB 20GiB
```

含义：

```
primary
```

分区类型（GPT 下仅作为名称保留）。

```
ext4
```

预期文件系统类型（只是标记，不会真正格式化）。

```
1MiB
```

起始位置。

```
20GiB
```

结束位置。

查看：

```
print
```

---

## 创建多个分区

例如：

```
mkpart primary ext4 1MiB 20GiB

mkpart primary ext4 20GiB 60GiB

mkpart primary xfs 60GiB 100%
```

其中

```
100%
```

表示磁盘末尾。

---

# 六、删除分区

查看：

```
print
```

例如：

```
Number
1
2
3
```

删除：

```
rm 2
```

删除第二个分区。

---

# 七、调整分区大小

查看帮助：

```
help resizepart
```

例如：

```
resizepart 2 100%
```

表示：

把第二个分区扩展到磁盘末尾。

注意：

**parted 只调整分区，不调整文件系统。**

例如：

EXT4：

```bash
resize2fs /dev/sdb2
```

XFS：

```bash
xfs_growfs /mountpoint
```

---

# 八、设置启动标志

例如 EFI 分区：

```
set 1 esp on
```

查看：

```
print
```

输出：

```
Flags: boot, esp
```

关闭：

```
set 1 esp off
```

---

# 九、查看帮助

交互模式：

```
help
```

查看某命令：

```
help mkpart
```

---

# 十、退出

```
quit
```

---

# 十一、非交互模式

很多自动化脚本都会这样写：

创建 GPT

```bash
parted -s /dev/sdb mklabel gpt
```

创建分区

```bash
parted -s /dev/sdb mkpart primary ext4 1MiB 20GiB
```

查看

```bash
parted -s /dev/sdb print
```

其中：

```
-s
```

表示 **script mode**（脚本模式），不会进入交互界面。

---

# 十二、常见单位

`parted` 支持多种容量单位：

| 单位   | 说明          |
| ---- | ----------- |
| B    | 字节          |
| KB   | 十进制 KB      |
| MB   | 十进制 MB      |
| GB   | 十进制 GB      |
| TB   | 十进制 TB      |
| KiB  | 二进制 KiB     |
| MiB  | 二进制 MiB（推荐） |
| GiB  | 二进制 GiB（推荐） |
| 100% | 磁盘末尾        |

推荐使用 **MiB/GiB**，因为它们与磁盘扇区对齐更准确，可避免性能损失。

---

# 十三、典型使用流程

假设新增了一块磁盘 `/dev/sdb`，创建一个占满整盘的 ext4 分区：

```bash
# 1. 创建 GPT 分区表
parted -s /dev/sdb mklabel gpt

# 2. 创建一个分区
parted -s /dev/sdb mkpart primary ext4 1MiB 100%

# 3. 格式化文件系统
mkfs.ext4 /dev/sdb1

# 4. 创建挂载点
mkdir -p /data

# 5. 挂载
mount /dev/sdb1 /data

# 6. 查看结果
lsblk
df -h
```

---

# 十四、与 fdisk 的比较

| 特性          | parted          | fdisk            |
| ----------- | --------------- | ---------------- |
| 支持 GPT      | ✔               | ✔（新版 util-linux） |
| 支持大于 2TB 磁盘 | ✔               | ✔（新版）            |
| 支持脚本模式      | ✔（`-s`）         | 有限               |
| 可调整分区大小     | ✔（`resizepart`） | ✘                |
| 操作界面        | 命令式             | 菜单式              |
| 适合自动化       | ✔               | 一般               |

**使用建议：**

* **新建 GPT 分区、管理大容量磁盘或编写自动化脚本**：优先使用 `parted`。
* **传统 MBR 磁盘或简单交互式分区**：`fdisk` 仍然是一个不错的选择。
* **调整文件系统容量**：记住 `parted` 只修改分区边界，完成后还需要根据文件系统类型使用 `resize2fs`（ext 系列）或 `xfs_growfs`（XFS）等工具扩展文件系统。
