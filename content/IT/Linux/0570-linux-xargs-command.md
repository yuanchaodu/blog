---
title: Linux xargs 命令
section: IT
category: Linux
---

# Linux xargs 命令

<img src="images/Linux.svg" width="300">

`xargs` 用来把**前一个命令的输出**，整理成**后一个命令的参数**。

最常见格式：

```bash
前一个命令 | xargs 后一个命令
```

例如：

```bash
echo "a b c" | xargs echo
```

等价于：

```bash
echo a b c
```

## 常用场景

### 1. 删除一批文件

```bash
find . -name "*.log" | xargs rm
```

含义：找到当前目录下所有 `.log` 文件，然后交给 `rm` 删除。

更安全写法：

```bash
find . -name "*.log" -print0 | xargs -0 rm
```

可以正确处理文件名中有空格、换行等特殊字符的情况。

### 2. 一次传多个参数

```bash
cat files.txt | xargs cat
```

如果 `files.txt` 内容是：

```text
a.txt
b.txt
c.txt
```

就相当于：

```bash
cat a.txt b.txt c.txt
```

### 3. 每次只处理一个参数

```bash
cat files.txt | xargs -n 1 echo
```

含义：每次只传 1 个参数给 `echo`。

### 4. 指定参数位置

```bash
cat files.txt | xargs -I {} cp {} /backup/
```

含义：把每个文件复制到 `/backup/` 目录。

这里 `{}` 是占位符。

### 5. 并发执行

```bash
cat urls.txt | xargs -n 1 -P 4 curl -O
```

含义：每次处理 1 个 URL，同时最多并发 4 个任务。

## 常用选项

| 选项      | 作用                        |
| ------- | ------------------------- |
| `-n N`  | 每次传 N 个参数                 |
| `-I {}` | 使用占位符，指定参数位置              |
| `-0`    | 按空字符分隔，常配合 `find -print0` |
| `-P N`  | 并发执行 N 个进程                |
| `-p`    | 执行前询问确认                   |
| `-t`    | 执行前打印实际命令                 |

## 记忆方式

`xargs` 可以理解成：

> 把管道里的文本，变成命令后面的参数。

比如：

```bash
ls *.txt | xargs rm
```

就是把：

```text
a.txt b.txt c.txt
```

变成：

```bash
rm a.txt b.txt c.txt
```

实际使用中，处理文件名时建议优先使用：

```bash
find . -type f -print0 | xargs -0 命令
```