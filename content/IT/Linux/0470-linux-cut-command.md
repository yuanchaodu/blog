---
title: Linux cut 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnFJg
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/47'
---

# Linux cut 命令

<img src="images/Linux.svg" width="300">

`cut` 是 Linux 中用于**按列提取文本内容**的命令，常用于处理日志、配置文件、CSV 文件等。

## 基本语法

```bash
cut [选项] 文件名
```

常用选项：

| 选项             | 说明         |
| -------------- | ---------- |
| `-b`           | 按字节提取      |
| `-c`           | 按字符提取      |
| `-d`           | 指定字段分隔符    |
| `-f`           | 提取指定字段     |
| `--complement` | 取反，提取未指定部分 |

---

## 1. 按字符提取

### 提取第1个字符

```bash
echo "abcdefg" | cut -c 1
```

输出：

```text
a
```

### 提取第1-3个字符

```bash
echo "abcdefg" | cut -c 1-3
```

输出：

```text
abc
```

### 提取多个位置

```bash
echo "abcdefg" | cut -c 1,3,5
```

输出：

```text
ace
```

---

## 2. 按字段提取（最常用）

假设有文件：

```text
Tom:18:Beijing
Jack:20:Shanghai
Lucy:22:Shenzhen
```

### 提取第1列

```bash
cut -d ":" -f 1 test.txt
```

输出：

```text
Tom
Jack
Lucy
```

---

### 提取第2列

```bash
cut -d ":" -f 2 test.txt
```

输出：

```text
18
20
22
```

---

### 提取多列

```bash
cut -d ":" -f 1,3 test.txt
```

输出：

```text
Tom:Beijing
Jack:Shanghai
Lucy:Shenzhen
```

---

### 提取连续列

```bash
cut -d ":" -f 2-3 test.txt
```

输出：

```text
18:Beijing
20:Shanghai
22:Shenzhen
```

---

## 3. 处理 `/etc/passwd`

Linux 中最经典的示例：

```bash
cat /etc/passwd
```

内容类似：

```text
root:x:0:0:root:/root:/bin/bash
mysql:x:27:27:MySQL Server:/var/lib/mysql:/sbin/nologin
```

### 提取用户名

```bash
cut -d ":" -f 1 /etc/passwd
```

输出：

```text
root
mysql
...
```

---

### 提取用户登录 Shell

```bash
cut -d ":" -f 7 /etc/passwd
```

输出：

```text
/bin/bash
/sbin/nologin
```

---

## 4. 配合其他命令使用

### 查看当前登录用户

```bash
who | cut -d " " -f 1
```

更可靠的方法：

```bash
who | cut -c 1-8
```

---

### 查看磁盘使用率

```bash
df -h | tail -n +2 | cut -d " " -f 1
```

但由于多个空格容易导致字段不准确，实际推荐：

```bash
df -h | awk '{print $1}'
```

---

## 5. 指定输出非某些字段

例如：

```text
Tom:18:Beijing:IT
```

删除第2列：

```bash
echo "Tom:18:Beijing:IT" | cut -d ":" -f 2 --complement
```

输出：

```text
Tom:Beijing:IT
```

---

## 6. 从 CSV 文件中提取

文件：

```text
name,age,city
Tom,18,Beijing
Jack,20,Shanghai
```

提取姓名：

```bash
cut -d "," -f 1 data.csv
```

输出：

```text
name
Tom
Jack
```

---

## cut 与 awk 的区别

| 命令     | 特点                |
| ------ | ----------------- |
| `cut`  | 简单、速度快，适合固定字段提取   |
| `awk`  | 功能强大，可计算、判断、格式化输出 |
| `sed`  | 擅长文本替换            |
| `grep` | 擅长文本过滤            |

例如：

```bash
cut -d ":" -f 1 /etc/passwd
```

等价于：

```bash
awk -F ":" '{print $1}' /etc/passwd
```

但如果需要条件判断：

```bash
awk -F ":" '$3>=1000 {print $1}'
```

则 `cut` 无法实现。

---

## 常用实例速查

```bash
# 提取第1列
cut -d ":" -f 1 file

# 提取第1和第3列
cut -d ":" -f 1,3 file

# 提取第2到第5列
cut -d ":" -f 2-5 file

# 提取第1个字符
cut -c 1 file

# 提取第1到10个字符
cut -c 1-10 file

# 删除第2列
cut -d ":" -f 2 --complement file

# 提取CSV第一列
cut -d "," -f 1 data.csv
```

### 注意事项

1. `cut -f` 必须配合字段分隔符 `-d` 使用（默认分隔符为 Tab）。
2. `cut` 只能处理简单字段提取，不能进行计算和条件判断。
3. 对于多个连续空格分隔的数据，`cut` 不太适用，通常使用 `awk` 更方便。

例如：

```bash
ps -ef | awk '{print $2}'
```

通常比：

```bash
ps -ef | cut ...
```

更可靠。
