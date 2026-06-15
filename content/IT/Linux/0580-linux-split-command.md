---
title: Linux split 命令
section: IT
category: Linux
---

# Linux split 命令

<img src="images/Linux.svg" width="300">

`split` 是 Linux 里用来**把大文件切成多个小文件**的命令，常用于日志、备份包、数据文件分割。

## 基本格式

```bash
split [选项] 原文件 输出文件前缀
```

例如：

```bash
split big.log part_
```

会生成：

```bash
part_aa
part_ab
part_ac
...
```

## 常用用法

按大小切分：

```bash
split -b 100M big.zip part_
```

表示每个文件最大 100MB。

按行数切分：

```bash
split -l 10000 big.log part_
```

表示每 10000 行切成一个文件。

使用数字后缀：

```bash
split -d -b 100M big.zip part_
```

生成：

```bash
part_00
part_01
part_02
...
```

指定后缀长度：

```bash
split -d -a 3 -b 100M big.zip part_
```

生成：

```bash
part_000
part_001
part_002
...
```

## 合并文件

如果是普通按大小切分的文件，可以用：

```bash
cat part_* > big.zip
```

注意文件顺序要正确。

## 实用示例

切分 2GB 文件，每个 500MB：

```bash
split -b 500M data.tar.gz data_part_
```

再合并：

```bash
cat data_part_* > data.tar.gz
```

一句话：`split` 负责“切”，`cat` 负责“拼”。