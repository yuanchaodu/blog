---
title: Linux cp 命令
section: IT
category: Linux
---

# Linux cp 命令

<img src="images/Linux.svg" width="300">

`cp` 是 Linux 中用于**复制文件或目录**的命令。

## 基本格式

```bash
cp [选项] 源文件 目标位置
```

## 常用示例

复制文件到目录：

```bash
cp a.txt /home/user/
```

复制并改名：

```bash
cp a.txt b.txt
```

复制目录：

```bash
cp -r dir1 dir2
```

覆盖前询问：

```bash
cp -i a.txt b.txt
```

显示复制过程：

```bash
cp -v a.txt /tmp/
```

保留文件属性，如权限、时间：

```bash
cp -p a.txt /backup/
```

复制多个文件到目录：

```bash
cp a.txt b.txt /tmp/
```

## 常用选项

| 选项          | 作用           |
| ----------- | ------------ |
| `-r` 或 `-R` | 递归复制目录       |
| `-i`        | 覆盖前确认        |
| `-f`        | 强制覆盖         |
| `-v`        | 显示过程         |
| `-p`        | 保留权限、时间等属性   |
| `-a`        | 归档复制，常用于备份目录 |

## 常见用法

备份配置文件：

```bash
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

复制整个目录并保留属性：

```bash
cp -a /data/app /backup/
```