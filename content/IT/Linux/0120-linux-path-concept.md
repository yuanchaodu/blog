---
title: Linux 绝对路径与相对路径
section: IT
category: Linux
---

# Linux 绝对路径与相对路径

<img src="images/Linux.svg" width="300">

在 Linux 中，**路径**就是文件或目录的位置。

## 1. 绝对路径

**绝对路径**从根目录 `/` 开始写，位置固定，不受当前目录影响。

例子：

```bash
/home/nick/test.txt
/etc/nginx/nginx.conf
/var/log/syslog
```

特点：

```bash
cd /home/nick
cat /etc/hosts
```

无论你当前在哪个目录，这些路径都能准确找到目标。

## 2. 相对路径

**相对路径**从“当前所在目录”开始写，不以 `/` 开头。

例子：

```bash
test.txt
./test.txt
../data/file.txt
docs/readme.md
```

常见符号：

```bash
.   表示当前目录
..  表示上一级目录
~   表示当前用户的家目录
```

注意：`~` 虽然不是从 `/` 开始，但通常会被 Shell 展开成绝对路径，比如：

```bash
~/test.txt
```

等价于：

```bash
/home/nick/test.txt
```

## 3. 简单例子

假设当前目录是：

```bash
/home/nick/project
```

那么：

```bash
./a.txt
```

表示：

```bash
/home/nick/project/a.txt
```

而：

```bash
../a.txt
```

表示：

```bash
/home/nick/a.txt
```

## 4. 判断方法

看路径开头：

```bash
/开头   绝对路径
非/开头 相对路径
```

一句话记忆：

**绝对路径像完整地址，相对路径像“从我这里往哪走”。**