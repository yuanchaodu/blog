---
title: Linux uniq 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnHxd
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/50'
---

# Linux uniq 命令

<img src="images/Linux.svg" width="300">

`uniq` 是 Linux 中用于**去除相邻重复行**的命令，通常与 `sort` 配合使用，用来统计、筛选或删除重复数据。

## 基本语法

```bash
uniq [选项] 文件
```

或者：

```bash
cat file.txt | uniq
```

## 工作原理

`uniq` 只会检查**相邻的行**是否重复。

例如文件内容：

```text
apple
apple
banana
orange
orange
orange
```

执行：

```bash
uniq file.txt
```

输出：

```text
apple
banana
orange
```

---

## 为什么经常和 sort 一起使用

如果重复行不相邻，`uniq` 无法识别。

例如：

```text
apple
banana
apple
orange
banana
```

执行：

```bash
uniq file.txt
```

输出不变：

```text
apple
banana
apple
orange
banana
```

正确做法：

```bash
sort file.txt | uniq
```

输出：

```text
apple
banana
orange
```

---

## 常用选项

### 1. -c 统计重复次数

统计每行出现次数：

```bash
sort file.txt | uniq -c
```

输出：

```text
      2 apple
      2 banana
      1 orange
```

---

### 2. -d 仅显示重复行

```bash
sort file.txt | uniq -d
```

输出：

```text
apple
banana
```

---

### 3. -u 仅显示唯一行

```bash
sort file.txt | uniq -u
```

输出：

```text
orange
```

---

### 4. -i 忽略大小写

文件：

```text
Apple
apple
APPLE
banana
```

执行：

```bash
sort file.txt | uniq -i
```

输出：

```text
Apple
banana
```

---

### 5. --all-repeated 显示所有重复组

```bash
uniq --all-repeated file.txt
```

适用于查看重复数据分组情况。

---

## 实际应用示例

### 统计日志中 IP 出现次数

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

说明：

```text
awk       提取IP
sort      排序
uniq -c   统计次数
sort -nr  按数量倒序排列
```

结果：

```text
125 192.168.1.10
 86 192.168.1.20
 34 192.168.1.30
```

---

### 查找重复用户名

```bash
cut -d: -f1 users.txt | sort | uniq -d
```

---

### 统计目录下文件扩展名

```bash
ls | awk -F. '{print $NF}' | sort | uniq -c
```

输出：

```text
10 txt
 5 log
 3 sh
```

---

## uniq 与 sort、wc 组合

统计唯一行数量：

```bash
sort file.txt | uniq | wc -l
```

或者更简单：

```bash
sort file.txt | uniq -c
```

---

## 与 sort -u 的区别

下面两个命令结果类似：

```bash
sort file.txt | uniq
```

```bash
sort -u file.txt
```

区别：

* `sort -u`：排序时直接去重，效率更高。
* `uniq`：除了去重，还能统计次数（`-c`）、只显示重复项（`-d`）、只显示唯一项（`-u`）等。

因此：

* 仅去重：推荐 `sort -u`
* 需要统计或分析重复情况：使用 `uniq`

## 总结

| 命令             | 作用        |
| -------------- | --------- |
| `uniq`         | 删除相邻重复行   |
| `uniq -c`      | 统计每行出现次数  |
| `uniq -d`      | 只显示重复行    |
| `uniq -u`      | 只显示唯一行    |
| `uniq -i`      | 忽略大小写     |
| `sort \| uniq` | 全局去重（最常见） |
| `sort -u`      | 排序并去重     |

记住一句话：**`uniq` 只能处理相邻重复行，因此绝大多数场景下都要先用 `sort` 排序。**
