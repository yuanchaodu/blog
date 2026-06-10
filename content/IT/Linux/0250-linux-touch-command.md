---
title: Linux touch 命令
section: IT
category: Linux
---

# Linux touch 命令

<img src="images/Linux.svg" width="300">

`touch` 是 Linux 中一个非常常用的命令，主要有两个作用：

1. **创建空文件**
2. **修改文件的时间戳（访问时间和修改时间）**

---

## 基本语法

```bash
touch [选项] 文件名
```

---

## 1. 创建新文件

如果文件不存在，`touch` 会创建一个空文件。

例如：

```bash
touch test.txt
```

执行后会创建：

```text
test.txt
```

查看：

```bash
ls -l test.txt
```

---

## 2. 修改文件时间

如果文件已经存在，`touch` 不会修改文件内容，只会更新文件时间。

例如：

```bash
touch test.txt
```

执行后：

* Access Time（访问时间）
* Modify Time（修改时间）

会更新为当前时间。

查看时间：

```bash
ls -l test.txt
```

或者：

```bash
stat test.txt
```

输出类似：

```text
Access: 2026-06-10 10:20:00
Modify: 2026-06-10 10:20:00
Change: 2026-06-10 10:20:00
```

---

## 常用选项

### -a 只修改访问时间

```bash
touch -a test.txt
```

仅更新 Access Time。

---

### -m 只修改修改时间

```bash
touch -m test.txt
```

仅更新 Modify Time。

---

### -c 不创建文件

如果文件不存在，不创建。

```bash
touch -c test.txt
```

常用于脚本中避免误创建文件。

---

### -t 指定时间

格式：

```bash
touch -t [[CC]YY]MMDDhhmm[.ss] 文件
```

例如：

```bash
touch -t 202506101530 test.txt
```

表示：

```text
2025-06-10 15:30:00
```

查看：

```bash
stat test.txt
```

---

### -d 指定日期时间

更加直观。

例如：

```bash
touch -d "2025-06-10 15:30:00" test.txt
```

或者：

```bash
touch -d "yesterday" test.txt
touch -d "2 days ago" test.txt
touch -d "next week" test.txt
```

---

### -r 参考其他文件时间

让目标文件继承参考文件的时间戳。

```bash
touch -r file1.txt file2.txt
```

执行后：

```text
file2.txt 的时间 = file1.txt 的时间
```

---

## 批量创建文件

创建多个文件：

```bash
touch file1.txt file2.txt file3.txt
```

---

## 创建连续编号文件

```bash
touch file{1..5}.txt
```

结果：

```text
file1.txt
file2.txt
file3.txt
file4.txt
file5.txt
```

---

## 实际应用场景

### 场景1：生成日志文件

```bash
touch app.log
```

---

### 场景2：测试目录结构

```bash
mkdir project
cd project

touch README.md
touch config.yaml
touch main.py
```

快速搭建项目骨架。

---

### 场景3：触发程序重新加载

有些程序会监控文件修改时间：

```bash
touch nginx.conf
```

相当于告诉程序：

```text
这个文件刚刚被修改过
```

---

## 与 echo 的区别

创建空文件：

```bash
touch test.txt
```

创建带内容文件：

```bash
echo "Hello" > test.txt
```

区别：

| 命令                    | 文件不存在     | 文件已存在 |
| --------------------- | --------- | ----- |
| touch test.txt        | 创建空文件     | 更新时间  |
| echo "abc" > test.txt | 创建文件并写入内容 | 覆盖原内容 |

---

## 总结

最常见的几个用法：

```bash
# 创建空文件
touch file.txt

# 创建多个文件
touch a.txt b.txt c.txt

# 创建连续文件
touch file{1..10}.txt

# 修改时间为当前时间
touch file.txt

# 指定时间
touch -d "2025-06-10 15:30:00" file.txt

# 复制另一个文件的时间
touch -r source.txt target.txt

# 文件不存在时不创建
touch -c file.txt
```

对于日常运维和脚本编写，最常见的就是：

```bash
touch 文件名
```

既能创建文件，又能更新时间戳。