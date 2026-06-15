---
title: Linux tr 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnIrd
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/56'
---

# Linux tr 命令

<img src="images/Linux.svg" width="300">

`tr` 是 Linux 中用来**转换、删除、压缩字符**的命令，常用于处理文本流。

## 基本格式

```bash
tr [选项] 字符集1 [字符集2]
```

它通常配合管道使用：

```bash
echo "hello" | tr 'a-z' 'A-Z'
```

输出：

```text
HELLO
```

## 常见用法

### 1. 小写转大写

```bash
echo "linux" | tr 'a-z' 'A-Z'
```

输出：

```text
LINUX
```

### 2. 大写转小写

```bash
echo "HELLO" | tr 'A-Z' 'a-z'
```

### 3. 删除字符

删除数字：

```bash
echo "abc123" | tr -d '0-9'
```

输出：

```text
abc
```

### 4. 压缩重复字符

把多个空格压成一个空格：

```bash
echo "a    b     c" | tr -s ' '
```

输出：

```text
a b c
```

### 5. 替换换行符

把多行合成一行：

```bash
cat file.txt | tr '\n' ' '
```

## 常用选项

| 选项   | 含义                 |
| ---- | ------------------ |
| `-d` | 删除指定字符             |
| `-s` | 压缩连续重复字符           |
| `-c` | 取反，表示“除了这些字符”      |
| `-t` | 截断字符集1，使其长度和字符集2一致 |

## 常见字符集

```bash
[:lower:]   小写字母
[:upper:]   大写字母
[:digit:]   数字
[:space:]   空白字符
[:alpha:]   字母
[:alnum:]   字母和数字
```

示例：

```bash
echo "Hello 123" | tr '[:lower:]' '[:upper:]'
```

输出：

```text
HELLO 123
```

## 典型场景

清理文本中的特殊字符，只保留字母和数字：

```bash
echo "a@b#c123!" | tr -cd '[:alnum:]'
```

输出：

```text
abc123
```

`tr` 适合做简单字符级处理；如果要按字段、行、正则规则处理文本，通常用 `awk`、`sed` 更合适。
