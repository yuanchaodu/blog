---
title: Linux awk 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnIqd
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/55'
---

# Linux awk 命令

<img src="images/Linux.svg" width="300">

`awk` 是 Linux 中常用的**文本处理命令**，特别适合按列处理日志、表格、配置文件等内容。

## 基本格式

```bash
awk '条件 {动作}' 文件名
```

例如：

```bash
awk '{print $1}' file.txt
```

表示打印 `file.txt` 每一行的第 1 列。

## 常用示例

### 1. 打印整行

```bash
awk '{print $0}' file.txt
```

`$0` 表示当前整行。

### 2. 打印指定列

```bash
awk '{print $1, $3}' file.txt
```

打印第 1 列和第 3 列。

### 3. 指定分隔符

默认按空格或 Tab 分列。

```bash
awk -F: '{print $1}' /etc/passwd
```

`-F:` 表示用冒号作为分隔符。

### 4. 按条件筛选

```bash
awk '$3 > 100 {print $1, $3}' file.txt
```

表示第 3 列大于 100 时，打印第 1 列和第 3 列。

### 5. 打印行号

```bash
awk '{print NR, $0}' file.txt
```

`NR` 表示当前行号。

### 6. 统计求和

```bash
awk '{sum += $2} END {print sum}' file.txt
```

统计第 2 列的总和。

## 常用内置变量

| 变量    | 含义      |
| ----- | ------- |
| `$0`  | 当前整行    |
| `$1`  | 第 1 列   |
| `$2`  | 第 2 列   |
| `NR`  | 当前行号    |
| `NF`  | 当前行字段数量 |
| `FS`  | 输入分隔符   |
| `OFS` | 输出分隔符   |

## 常用场景

查看日志中某一列：

```bash
awk '{print $1}' access.log
```

统计访问 IP：

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

查看磁盘使用率：

```bash
df -h | awk 'NR>1 {print $1, $5}'
```

一句话记忆：`awk` 适合“按行读取、按列处理、按条件输出”。
