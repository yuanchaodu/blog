---
title: Linux sed 命令
section: IT
category: Linux
---

# Linux sed 命令

<img src="images/Linux.svg" width="300">

`sed` 是 Linux 中常用的**文本处理命令**，主要用于按行查找、替换、删除、插入文本。它适合处理配置文件、日志文件、脚本文件等。

## 常用格式

```bash
sed [选项] '命令' 文件名
```

例如：

```bash
sed 's/old/new/' file.txt
```

表示把每一行中第一个 `old` 替换成 `new`。

## 常用操作

### 1. 替换文本

```bash
sed 's/apple/orange/' file.txt
```

只替换每行第一个匹配项。

替换每行所有匹配项：

```bash
sed 's/apple/orange/g' file.txt
```

直接修改原文件：

```bash
sed -i 's/apple/orange/g' file.txt
```

### 2. 删除行

删除第 3 行：

```bash
sed '3d' file.txt
```

删除第 3 到第 5 行：

```bash
sed '3,5d' file.txt
```

删除包含 `error` 的行：

```bash
sed '/error/d' file.txt
```

### 3. 打印指定行

打印第 5 行：

```bash
sed -n '5p' file.txt
```

打印第 5 到第 10 行：

```bash
sed -n '5,10p' file.txt
```

### 4. 在行前或行后插入内容

在第 3 行前插入：

```bash
sed '3i\这是新内容' file.txt
```

在第 3 行后追加：

```bash
sed '3a\这是新内容' file.txt
```

### 5. 替换整行

把第 3 行替换为新内容：

```bash
sed '3c\这是替换后的内容' file.txt
```

## 常见选项

```bash
-n   只输出被处理的行
-i   直接修改原文件
-e   执行多个 sed 命令
```

多个命令示例：

```bash
sed -e 's/apple/orange/g' -e '/error/d' file.txt
```

## 工作中常用例子

修改配置文件中的端口：

```bash
sed -i 's/port=8080/port=9090/' app.conf
```

删除空行：

```bash
sed '/^$/d' file.txt
```

删除注释行：

```bash
sed '/^#/d' file.txt
```

查看日志中包含关键字的行：

```bash
sed -n '/ERROR/p' app.log
```

`sed` 可以理解为一个“批量文本编辑器”，特别适合在脚本中自动处理文件。