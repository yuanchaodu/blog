---
title: Linux cpio 命令
section: IT
category: Linux
---

# Linux cpio 命令

<img src="images/Linux.svg" width="300">

`cpio`（Copy In/Out）是 Linux/Unix 系统中用于创建、提取和复制归档文件的经典工具。它经常用于制作 Linux 启动镜像中的 **initramfs**，也常与 `find` 命令配合使用进行文件备份和恢复。

## 一、cpio 的工作模式

`cpio` 有三种主要工作模式：

| 模式     | 参数   | 作用           |
| ------ | ---- | ------------ |
| 归档（创建） | `-o` | 创建 cpio 归档文件 |
| 解包（提取） | `-i` | 从归档文件中提取文件   |
| 复制（传输） | `-p` | 在目录间复制文件     |

---

## 二、创建归档文件

### 示例1：打包当前目录所有文件

```bash
find . | cpio -o > archive.cpio
```

说明：

```text
find .      # 查找所有文件
cpio -o     # 创建归档
> archive.cpio
```

---

### 示例2：使用 newc 格式（最常用）

Linux initramfs 通常使用 `newc` 格式：

```bash
find . | cpio -o -H newc > initramfs.cpio
```

参数说明：

| 参数      | 说明           |
| ------- | ------------ |
| -H newc | 指定归档格式为 newc |
| -o      | 输出归档         |

---

### 示例3：创建压缩归档

```bash
find . | cpio -o -H newc | gzip > backup.cpio.gz
```

或者：

```bash
find . | cpio -o -H newc | xz > backup.cpio.xz
```

---

## 三、查看归档内容

### 查看文件列表

```bash
cpio -it < archive.cpio
```

或：

```bash
gzip -dc backup.cpio.gz | cpio -it
```

参数：

```text
-i  提取模式
-t  只显示文件名
```

输出示例：

```text
.
./etc
./etc/passwd
./bin
./bin/bash
```

---

## 四、提取归档文件

### 解压全部内容

```bash
cpio -id < archive.cpio
```

参数：

| 参数 | 说明     |
| -- | ------ |
| -i | 提取     |
| -d | 自动创建目录 |

---

### 保留原文件时间

```bash
cpio -idm < archive.cpio
```

其中：

```text
-m  保留文件修改时间
```

---

### 提取指定文件

```bash
cpio -id "etc/passwd" < archive.cpio
```

或者：

```bash
cpio -id "./etc/*" < archive.cpio
```

---

## 五、目录复制模式

这是 `cpio` 一个比较特殊但很实用的功能。

### 复制目录树

```bash
find . -depth | cpio -pdm /backup
```

参数说明：

| 参数 | 作用             |
| -- | -------------- |
| -p | Pass-through模式 |
| -d | 自动创建目录         |
| -m | 保留时间           |

效果类似：

```bash
cp -a source/* /backup/
```

但在处理大量文件时效率较高。

---

## 六、常用参数

| 参数      | 含义     |
| ------- | ------ |
| -o      | 创建归档   |
| -i      | 提取归档   |
| -p      | 目录复制   |
| -d      | 创建目录   |
| -m      | 保留时间   |
| -v      | 显示详细过程 |
| -t      | 查看列表   |
| -u      | 覆盖已有文件 |
| -H newc | newc格式 |
| -H crc  | 带CRC校验 |
| --quiet | 静默模式   |

---

## 七、initramfs 中的典型应用

查看当前系统 initramfs 内容：

```bash
mkdir tmp
cd tmp

zcat /boot/initramfs-$(uname -r).img | cpio -idmv
```

查看目录：

```bash
ls
```

通常会看到：

```text
bin/
dev/
etc/
init
lib/
proc/
sys/
usr/
```

---

## 八、与 tar 的区别

| 对比项        | cpio        | tar  |
| ---------- | ----------- | ---- |
| 输入方式       | 从标准输入读取文件列表 | 指定目录 |
| 与 find 配合  | 非常方便        | 一般   |
| initramfs  | 广泛使用        | 很少   |
| 增量备份       | 支持          | 支持   |
| 使用普及度      | 较低          | 非常高  |
| Linux发行版打包 | 很少          | 常见   |

例如：

```bash
find /etc -name "*.conf" | cpio -o > conf.cpio
```

这种按条件筛选后再打包，是 `cpio` 最大的优势。

---

## 九、实际运维常见用法

### 备份配置文件

```bash
find /etc -name "*.conf" | cpio -o -H newc > etc_conf.cpio
```

### 恢复配置文件

```bash
cpio -idmv < etc_conf.cpio
```

### 打包指定文件列表

```bash
cat filelist.txt
```

内容：

```text
/etc/passwd
/etc/group
/etc/hosts
```

打包：

```bash
cat filelist.txt | cpio -o > config.cpio
```

---

## 十、查看 cpio 文件格式

```bash
file archive.cpio
```

输出示例：

```text
archive.cpio: ASCII cpio archive (SVR4 with no CRC)
```

常见格式：

* bin（旧格式）
* odc（兼容格式）
* newc（Linux最常用）
* crc（带校验）

现代 Linux 环境中，特别是在制作和分析 **initramfs** 时，最常见的命令组合是：

```bash
find . | cpio -o -H newc | gzip > initramfs.cpio.gz
```

以及解包：

```bash
gzip -dc initramfs.cpio.gz | cpio -idmv
```

这是嵌入式 Linux、内核开发和系统启动镜像处理中最常见的用法。