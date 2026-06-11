---
title: Linux cat 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDko
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/41'
---

# Linux cat 命令

<img src="images/Linux.svg" width="300">

`cat`（concatenate）是 Linux 中最常用的文本查看和文件处理命令之一，主要用于**查看文件内容、连接多个文件、创建文件以及输出内容到终端**。

## 一、基本语法

```bash
cat [选项] 文件名
```

例如：

```bash
cat test.txt
```

显示 `test.txt` 的全部内容。

---

## 二、常见用法

### 1. 查看文件内容

```bash
cat file.txt
```

输出文件全部内容。

示例：

```bash
$ cat hello.txt
Hello World
Linux Study
```

---

### 2. 查看多个文件

```bash
cat file1.txt file2.txt
```

按顺序输出多个文件内容。

例如：

```bash
cat a.txt b.txt
```

输出：

```text
a.txt内容
b.txt内容
```

---

### 3. 合并文件

将多个文件内容合并到新文件：

```bash
cat file1.txt file2.txt > file3.txt
```

此时：

```text
file3.txt = file1.txt + file2.txt
```

例如：

```bash
cat part1.txt part2.txt > all.txt
```

---

### 4. 创建文件

```bash
cat > test.txt
```

输入内容：

```text
hello
linux
```

按：

```text
Ctrl + D
```

结束输入并保存。

查看：

```bash
cat test.txt
```

输出：

```text
hello
linux
```

---

### 5. 追加内容

```bash
cat >> test.txt
```

输入：

```text
new line
```

按 `Ctrl+D` 保存。

原内容后面会增加：

```text
hello
linux
new line
```

---

## 三、常用选项

### -n 显示行号

```bash
cat -n file.txt
```

示例：

```text
     1  Hello
     2  Linux
     3  World
```

---

### -b 只给非空行编号

```bash
cat -b file.txt
```

示例：

```text
     1  Hello

     2  Linux
```

空行不会编号。

---

### -s 压缩连续空行

假设文件内容：

```text
Hello


Linux



World
```

执行：

```bash
cat -s file.txt
```

输出：

```text
Hello

Linux

World
```

连续多个空行压缩为一个。

---

### -E 显示行尾符号

```bash
cat -E file.txt
```

输出：

```text
Hello$
Linux$
World$
```

`$` 表示行结束位置。

---

### -T 显示 Tab

```bash
cat -T file.txt
```

Tab 会显示成：

```text
^I
```

例如：

```text
Name^IAge
Tom^I18
```

---

### -A 显示所有不可见字符

相当于：

```bash
cat -vET
```

例如：

```bash
cat -A file.txt
```

可以查看：

* Tab
* 换行
* 特殊控制字符

常用于排查编码和格式问题。

---

## 四、运维中的常见场景

### 查看配置文件

```bash
cat /etc/passwd
```

```bash
cat /etc/hosts
```

```bash
    cat /etc/resolv.conf
```

---

### 查看日志文件

小日志：

```bash
cat app.log
```

大日志不推荐：

```bash
cat huge.log
```

应使用：

```bash
less huge.log
```

或：

```bash
tail -f huge.log
```

---

### 查看系统信息

```bash
cat /proc/cpuinfo
```

查看 CPU 信息。

```bash
cat /proc/meminfo
```

查看内存信息。

```bash
cat /proc/version
```

查看内核版本。

---

## 五、与 less、more 的区别

| 命令   | 特点            |
| ---- | ------------- |
| cat  | 一次性全部输出       |
| more | 分页查看          |
| less | 分页查看，可前后翻页、搜索 |
| tail | 查看文件末尾        |
| head | 查看文件开头        |

例如查看 10GB 日志：

❌ 不建议：

```bash
cat huge.log
```

因为会一次性刷屏。

✅ 建议：

```bash
less huge.log
```

或者：

```bash
tail -100 huge.log
```

---

## 六、一个经典技巧

统计文件行数时经常看到：

```bash
cat file.txt | wc -l
```

实际上更推荐：

```bash
wc -l file.txt
```

因为：

```bash
cat file.txt | wc -l
```

会额外启动一个 `cat` 进程，效率较低。

这在 Linux 圈子里甚至有个专门的说法：

> Useless Use of Cat（无意义地使用 cat）

例如：

```bash
grep error file.log
```

比：

```bash
cat file.log | grep error
```

更简洁高效。

---

## 总结

`cat` 最常见的用途有四个：

```bash
# 查看文件
cat file.txt

# 查看多个文件
cat file1 file2

# 合并文件
cat file1 file2 > newfile

# 创建文件
cat > file.txt
```

对于系统管理员和运维工程师来说，`cat` 是查看配置文件、检查系统信息和快速处理文本文件时使用频率最高的基础命令之一。
