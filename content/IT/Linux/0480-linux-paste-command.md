---
title: Linux paste 命令
section: IT
category: Linux
---

# Linux paste 命令

<img src="images/Linux.svg" width="300">


`paste` 是 Linux 中用于**按列合并文件内容**的命令。它会将多个文件的对应行横向拼接，而不是像 `cat` 那样纵向连接。

## 基本语法

```bash
paste [选项] 文件1 文件2 ...
```

## 示例

### 1. 合并两个文件

文件 `a.txt`：

```text
A
B
C
```

文件 `b.txt`：

```text
1
2
3
```

执行：

```bash
paste a.txt b.txt
```

结果：

```text
A   1
B   2
C   3
```

默认使用 Tab 作为分隔符。

---

### 2. 指定分隔符

使用逗号：

```bash
paste -d "," a.txt b.txt
```

结果：

```text
A,1
B,2
C,3
```

使用多个分隔符：

```bash
paste -d ",:" a.txt b.txt c.txt
```

分隔符会循环使用。

---

### 3. 将单个文件转为一行

文件：

```text
A
B
C
D
```

执行：

```bash
paste -s a.txt
```

结果：

```text
A   B   C   D
```

`-s`（serial）表示串行处理。

---

### 4. 结合自定义分隔符转为 CSV

```bash
paste -s -d "," a.txt
```

结果：

```text
A,B,C,D
```

---

### 5. 从标准输入读取

```bash
ls | paste -s -d ","
```

结果类似：

```text
file1,file2,file3,file4
```

---

## 常用选项

| 选项        | 说明                         |
| --------- | -------------------------- |
| `-d LIST` | 指定分隔符                      |
| `-s`      | 串行模式，将一个文件多行合并成一行          |
| `-z`      | 以 NUL 字符作为行结束符（处理特殊文件名时有用） |

---

## 与 cat 的区别

假设：

```text
a.txt:      b.txt:
A           1
B           2
```

`cat a.txt b.txt`：

```text
A
B
1
2
```

`paste a.txt b.txt`：

```text
A   1
B   2
```

可以把 `cat` 理解为“上下拼接”，而 `paste` 理解为“左右拼接”。

---

## 实际应用

### 生成 CSV

```bash
paste -d "," name.txt age.txt > users.csv
```

结果：

```csv
Tom,20
Jerry,18
Alice,22
```

### 合并两列数据

```bash
cut -f1 file.txt > col1.txt
cut -f2 file.txt > col2.txt

paste col1.txt col2.txt
```

### 将多行转成一行

```bash
cat list.txt | paste -sd ","
```

结果：

```text
item1,item2,item3,item4
```

这是 Shell 脚本中非常常见的用法。