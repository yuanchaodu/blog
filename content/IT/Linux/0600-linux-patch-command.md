---
title: Linux patch 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnIuA
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/60'
---

# Linux patch 命令

<img src="images/Linux.svg" width="300">

`patch` 是 Linux 中用来**把差异文件应用到源码或文本文件上**的命令。常和 `diff` 配合使用。

## 1. 生成补丁

```bash
diff -u old.txt new.txt > change.patch
```

含义：

```text
old.txt：原文件
new.txt：修改后的文件
change.patch：生成的补丁文件
```

## 2. 应用补丁

```bash
patch old.txt < change.patch
```

执行后，`old.txt` 会被修改成接近 `new.txt` 的内容。

## 3. 给目录打补丁

如果补丁里包含路径，常用：

```bash
patch -p1 < change.patch
```

`-p1` 表示去掉路径中的第一级目录。

例如补丁路径是：

```text
a/src/main.c
```

使用 `-p1` 后会变成：

```text
src/main.c
```

## 4. 预检查，不真正修改

```bash
patch --dry-run -p1 < change.patch
```

建议正式打补丁前先执行一次。

## 5. 回滚补丁

```bash
patch -R -p1 < change.patch
```

`-R` 表示反向应用补丁，也就是撤销。

## 6. 常用参数

| 参数              | 作用        |
| --------------- | --------- |
| `-pN`           | 去掉路径前 N 层 |
| `--dry-run`     | 只检查，不修改文件 |
| `-R`            | 反向打补丁     |
| `-i file.patch` | 指定补丁文件    |
| `-b`            | 修改前备份原文件  |
| `--verbose`     | 显示详细信息    |

## 常用写法

```bash
patch --dry-run -p1 < fix.patch
patch -p1 < fix.patch
```

这是最常见、也比较稳妥的用法。
