---
title: Linux wc 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnIk2
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/51'
---

# Linux wc 命令

<img src="images/Linux.svg" width="300">

`wc`（Word Count）是 Linux 中用于统计文件内容的常用命令，可以统计：

* 行数（lines）
* 单词数（words）
* 字节数（bytes）
* 字符数（characters）

## 基本语法

```bash
wc [选项] 文件名
```

例如：

```bash
wc test.txt
```

输出：

```bash
10  50  300 test.txt
```

表示：

```text
10   行数
50   单词数
300  字节数
```

---

## 常用选项

### 统计行数

```bash
wc -l test.txt
```

输出：

```bash
10 test.txt
```

实际工作中经常用于统计日志条数：

```bash
wc -l access.log
```

---

### 统计单词数

```bash
wc -w test.txt
```

输出：

```bash
50 test.txt
```

---

### 统计字符数

```bash
wc -m test.txt
```

输出：

```bash
280 test.txt
```

对于 UTF-8 中文文件，字符数和字节数可能不同。

---

### 统计字节数

```bash
wc -c test.txt
```

输出：

```bash
300 test.txt
```

---

### 统计最长行长度

```bash
wc -L test.txt
```

输出：

```bash
80 test.txt
```

表示文件中最长的一行有 80 个字符。

---

## 多文件统计

```bash
wc file1.txt file2.txt
```

输出：

```bash
10 50 300 file1.txt
20 80 500 file2.txt
30 130 800 total
```

最后一行是总计。

---

## 与管道配合使用

### 统计目录下文件数量

```bash
ls | wc -l
```

例如输出：

```bash
25
```

表示当前目录有 25 个文件和目录项。

---

### 统计进程数量

```bash
ps -ef | wc -l
```

---

### 统计日志中 ERROR 数量

```bash
grep "ERROR" app.log | wc -l
```

---

## 实际运维中的典型用法

### 统计 CSV 数据记录数

```bash
wc -l data.csv
```

如果第一行为表头：

```bash
expr $(wc -l < data.csv) - 1
```

或者：

```bash
echo $(( $(wc -l < data.csv) - 1 ))
```

---

### 统计某类文件数量

```bash
find /data -name "*.txt" | wc -l
```

---

### 统计代码行数

```bash
find . -name "*.py" | xargs wc -l
```

统计总行数：

```bash
find . -name "*.py" | xargs cat | wc -l
```

或者：

```bash
find . -name "*.py" -exec cat {} \; | wc -l
```

---

## 常见技巧

### 只输出数字，不显示文件名

```bash
wc -l < test.txt
```

输出：

```bash
10
```

而不是：

```bash
10 test.txt
```

这种写法在 Shell 脚本中非常常见。

例如：

```bash
LINES=$(wc -l < test.txt)
echo "$LINES"
```

---

## 速查表

| 命令                      | 作用           |
| ----------------------- | ------------ |
| `wc file.txt`           | 统计行数、单词数、字节数 |
| `wc -l file.txt`        | 统计行数         |
| `wc -w file.txt`        | 统计单词数        |
| `wc -m file.txt`        | 统计字符数        |
| `wc -c file.txt`        | 统计字节数        |
| `wc -L file.txt`        | 统计最长行长度      |
| `ls \| wc -l`           | 统计目录项数量      |
| `grep xxx log \| wc -l` | 统计匹配记录数      |
| `wc -l < file.txt`      | 只输出数字        |

对于日常运维、日志分析和 Shell 脚本开发来说，`wc -l` 是使用频率最高的形式，通常用于统计记录条数、文件数和查询结果数量。
