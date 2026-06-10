---
title: Linux ls 命令
section: IT
category: Linux
---

# Linux ls 命令

<img src="images/Linux.svg" width="300">

`ls`（list）是 Linux 中最常用的命令之一，用于显示目录中的文件和子目录。

## 基本语法

```bash
ls [选项] [目录或文件]
```

例如：

```bash
ls
```

显示当前目录下的文件和目录。

```bash
ls /home
```

显示 `/home` 目录中的内容。

---

## 常用选项

### 1. `ls -l`：长格式显示

```bash
ls -l
```

输出示例：

```bash
-rw-r--r-- 1 root root 1024 Jun 10 08:30 test.txt
drwxr-xr-x 2 root root 4096 Jun 10 08:20 mydir
```

各字段含义：

| 字段             | 说明       |
| -------------- | -------- |
| `-rw-r--r--`   | 文件类型和权限  |
| `1`            | 硬链接数     |
| `root`         | 所有者      |
| `root`         | 所属组      |
| `1024`         | 文件大小（字节） |
| `Jun 10 08:30` | 修改时间     |
| `test.txt`     | 文件名      |

---

### 2. `ls -a`：显示隐藏文件

Linux 中以 `.` 开头的文件为隐藏文件。

```bash
ls -a
```

例如：

```bash
.
..
.bashrc
.profile
test.txt
```

---

### 3. `ls -la`：显示详细信息和隐藏文件

最常用组合之一：

```bash
ls -la
```

---

### 4. `ls -h`：人类可读格式显示文件大小

通常与 `-l` 配合：

```bash
ls -lh
```

显示：

```bash
-rw-r--r-- 1 root root 1.2K Jun 10 08:30 test.txt
-rw-r--r-- 1 root root 5.6M Jun 10 08:40 video.mp4
```

---

### 5. `ls -t`：按修改时间排序

```bash
ls -lt
```

最新修改的文件排在前面。

---

### 6. `ls -r`：反向排序

```bash
ls -lr
```

---

### 7. `ls -S`：按文件大小排序

```bash
ls -lS
```

大文件排在前面。

---

### 8. `ls -R`：递归显示

```bash
ls -R
```

显示当前目录及所有子目录中的内容。

---

## 常见使用示例

### 查看当前目录

```bash
ls
```

---

### 查看详细文件信息

```bash
ls -l
```

---

### 查看所有文件（包括隐藏文件）

```bash
ls -a
```

---

### 查看文件大小并以 KB、MB 显示

```bash
ls -lh
```

---

### 按时间排序查看最新文件

```bash
ls -lt
```

---

### 查看指定目录

```bash
ls -l /var/log
```

---

### 查看多个目录

```bash
ls /etc /home
```

---

## 文件类型标识

在 `ls -l` 输出的第一列，第一个字符表示文件类型：

| 标识  | 类型    |
| --- | ----- |
| `-` | 普通文件  |
| `d` | 目录    |
| `l` | 软链接   |
| `c` | 字符设备  |
| `b` | 块设备   |
| `p` | 管道文件  |
| `s` | 套接字文件 |

例如：

```bash
drwxr-xr-x
```

表示一个目录。

```bash
lrwxrwxrwx
```

表示一个软链接。

---

## 实际运维常用组合

查看目录大小、权限和时间：

```bash
ls -alh
```

查看最近修改的 10 个文件：

```bash
ls -lt | head
```

查看最大的 10 个文件：

```bash
ls -lS | head
```

递归查看目录结构：

```bash
ls -R
```

---

## 与 `ll` 的区别

很多 Linux 发行版中：

```bash
ll
```

实际上是下面命令的别名：

```bash
ls -alF
```

可以通过查看：

```bash
alias ll
```

确认定义。

因此：

```bash
ll
```

通常比：

```bash
ls
```

显示更多信息，包括权限、文件大小、隐藏文件等。

---

对于日常系统管理，最常用的几个组合基本就是：

```bash
ls -alh      # 查看详细信息
ls -lt       # 按时间排序
ls -lS       # 按大小排序
ls -R        # 递归查看
```