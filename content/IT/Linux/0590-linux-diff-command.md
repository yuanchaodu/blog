---
title: Linux diff 命令
section: IT
category: Linux
---

# Linux diff 命令

<img src="images/Linux.svg" width="300">

`diff` 用来比较两个文件或目录的差异。

## 基本用法

```bash
diff 文件1 文件2
```

例如：

```bash
diff old.txt new.txt
```

没有输出，表示两个文件内容相同；有输出，表示存在差异。

## 常用参数

```bash
diff -u old.txt new.txt
```

以统一格式显示差异，最常用，适合看代码改动。

```bash
diff -r dir1 dir2
```

递归比较两个目录。

```bash
diff -q file1 file2
```

只判断是否不同，不显示具体差异。

```bash
diff -i file1 file2
```

忽略大小写差异。

```bash
diff -w file1 file2
```

忽略所有空格差异。

```bash
diff -B file1 file2
```

忽略空白行差异。

## 输出怎么看

示例：

```bash
diff a.txt b.txt
```

输出：

```text
2c2
< hello
---
> hello world
```

含义是：

`2c2`：第一个文件第 2 行改成第二个文件第 2 行。
`<`：表示第一个文件中的内容。
`>`：表示第二个文件中的内容。

## 推荐格式

实际工作中建议用：

```bash
diff -u old.txt new.txt
```

输出更清楚，例如：

```text
-旧内容
+新内容
```

`-` 表示被删除的行，`+` 表示新增的行。