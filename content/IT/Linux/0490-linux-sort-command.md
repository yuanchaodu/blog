---
title: Linux sort 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnHxJ
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/49'
---

# Linux sort 命令

<img src="images/Linux.svg" width="300">

`sort` 是 Linux 中最常用的文本排序命令之一，用于对文件内容或标准输入的数据进行排序。

## 基本语法

```bash
sort [选项] 文件名
```

例如：

```bash
cat test.txt
```

内容如下：

```text
banana
apple
orange
pear
```

执行：

```bash
sort test.txt
```

输出：

```text
apple
banana
orange
pear
```

默认按照字典顺序升序排序。

---

## 常用选项

### 1. 倒序排序（-r）

```bash
sort -r test.txt
```

结果：

```text
pear
orange
banana
apple
```

---

### 2. 数值排序（-n）

假设文件内容：

```text
100
20
3
50
```

直接排序：

```bash
sort num.txt
```

结果：

```text
100
20
3
50
```

因为默认按字符排序。

使用数值排序：

```bash
sort -n num.txt
```

结果：

```text
3
20
50
100
```

---

### 3. 去重排序（-u）

文件内容：

```text
apple
banana
apple
orange
banana
```

执行：

```bash
sort -u test.txt
```

结果：

```text
apple
banana
orange
```

相当于：

```bash
sort test.txt | uniq
```

---

### 4. 指定分隔符（-t）

例如 CSV 文件：

```text
Tom,85
Jack,92
Lucy,78
```

按第二列成绩排序：

```bash
sort -t ',' -k2 -n score.csv
```

结果：

```text
Lucy,78
Tom,85
Jack,92
```

其中：

* `-t ','` 指定分隔符为逗号
* `-k2` 指定按第2列排序
* `-n` 按数字排序

---

### 5. 按指定字段排序（-k）

文件内容：

```text
Tom 85
Jack 92
Lucy 78
```

按成绩排序：

```bash
sort -k2 -n score.txt
```

结果：

```text
Lucy 78
Tom 85
Jack 92
```

---

### 6. 忽略大小写（-f）

文件内容：

```text
apple
Banana
Orange
pear
```

执行：

```bash
sort -f test.txt
```

结果：

```text
apple
Banana
Orange
pear
```

排序时不区分大小写。

---

### 7. 按月份排序（-M）

文件内容：

```text
Jan
Mar
Feb
Dec
```

执行：

```bash
sort -M month.txt
```

结果：

```text
Jan
Feb
Mar
Dec
```

按照月份顺序而不是字母顺序排序。

---

### 8. 输出到文件（-o）

```bash
sort test.txt -o result.txt
```

排序结果直接写入：

```text
result.txt
```

---

## 多字段排序

假设数据：

```text
Tom  IT   5000
Jack HR   6000
Lucy IT   4500
Rose HR   5500
```

先按部门排序，再按工资排序：

```bash
sort -k2,2 -k3,3n employee.txt
```

结果：

```text
Rose HR 5500
Jack HR 6000
Lucy IT 4500
Tom IT 5000
```

说明：

* `-k2,2`：仅按第2列（部门）排序
* `-k3,3n`：部门相同时按第3列数字排序

---

## 查看重复记录

统计日志中重复 IP：

```bash
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

含义：

1. 提取 IP
2. 排序
3. 统计重复次数
4. 按出现次数倒序排列

示例输出：

```text
150 192.168.1.10
120 192.168.1.20
 50 192.168.1.30
```

---

## 实用示例

### 找出文件中最大的数字

```bash
sort -n data.txt | tail -1
```

或者：

```bash
sort -nr data.txt | head -1
```

---

### 查看磁盘占用最大的目录

```bash
du -sh * | sort -hr
```

参数说明：

* `-h`：识别 KB、MB、GB
* `-r`：倒序

输出：

```text
10G data
2.5G backup
800M logs
```

---

## 常见参数速查表

| 参数   | 作用               |
| ---- | ---------------- |
| `-n` | 按数字排序            |
| `-r` | 倒序排序             |
| `-u` | 排序后去重            |
| `-f` | 忽略大小写            |
| `-M` | 按月份排序            |
| `-h` | 按人类可读单位排序（K/M/G） |
| `-t` | 指定字段分隔符          |
| `-k` | 指定排序字段           |
| `-o` | 输出到文件            |
| `-c` | 检查文件是否已排序        |

---

## 企业运维中最常见的用法

### 按 IP 统计访问量

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

### 查看 CPU 占用最高的进程

```bash
ps aux --sort=-%cpu | head
```

或者：

```bash
ps aux | sort -k3 -nr | head
```

### 查看内存占用最高的进程

```bash
ps aux | sort -k4 -nr | head
```

### 查看最大的文件

```bash
find . -type f -exec du -h {} + | sort -hr | head -20
```

这些都是 Linux 运维、日志分析和数据处理工作中非常高频的 `sort` 应用场景。
