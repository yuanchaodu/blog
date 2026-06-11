---
title: Linux ln 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDjo
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/40'
---

# Linux ln 命令

<img src="images/Linux.svg" width="300">

`ln` 命令用于在 Linux 中创建**链接**，相当于给文件或目录增加一个“入口”。

## 1. 硬链接

```bash
ln 源文件 链接名
```

示例：

```bash
ln a.txt b.txt
```

含义：`b.txt` 和 `a.txt` 指向同一个文件内容。删除其中一个，另一个仍可用。

特点：不能跨文件系统，通常不能给目录创建硬链接。

## 2. 软链接（符号链接）

```bash
ln -s 源文件或目录 链接名
```

示例：

```bash
ln -s /opt/app/config.yml config.yml
```

含义：`config.yml` 像快捷方式一样指向真实文件。

特点：可以跨文件系统，可以链接目录。源文件删除后，软链接会失效。

## 3. 常用参数

```bash
-s   创建软链接
-f   强制覆盖已有链接或文件
-n   处理目录软链接时不进入目录
-v   显示创建过程
```

常用组合：

```bash
ln -sfn /new/path app
```

常用于更新目录软链接，比如版本切换。

## 4. 查看链接

```bash
ls -l
```

软链接会显示类似：

```bash
app -> /opt/app-v2
```

硬链接可用：

```bash
ls -li
```

看 inode 是否相同。

## 5. 删除链接

```bash
rm 链接名
```

删除链接不会删除源文件，但注意不要写成：

```bash
rm -r 链接目录/
```

末尾带 `/` 时可能进入目标目录操作。
