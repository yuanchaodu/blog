---
title: Linux ncdu 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDjL
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/39'
---

# Linux ncdu 命令

<img src="images/Linux.svg" width="300">

`ncdu` 是 Linux 下查看磁盘空间占用的命令，名字来自 **NCurses Disk Usage**。它比 `du -sh *` 更直观，可以交互式查看哪个目录、哪个文件占空间最大。

## 1. 安装

Debian / Ubuntu：

```bash
sudo apt install ncdu
```

CentOS / RHEL / Rocky：

```bash
sudo yum install ncdu
```

或：

```bash
sudo dnf install ncdu
```

## 2. 基本用法

查看当前目录占用：

```bash
ncdu
```

查看指定目录：

```bash
ncdu /var
```

查看根目录：

```bash
sudo ncdu /
```

## 3. 常用操作键

进入目录：

```text
Enter
```

返回上级：

```text
← 或 h
```

删除选中的文件或目录：

```text
d
```

退出：

```text
q
```

按大小排序：

```text
s
```

按名称排序：

```text
n
```

显示帮助：

```text
?
```

## 4. 常用参数

不跨文件系统，适合查根目录时避免扫到挂载盘：

```bash
sudo ncdu -x /
```

排除某个目录：

```bash
ncdu --exclude /proc --exclude /sys /
```

导出扫描结果：

```bash
ncdu -o disk.json /
```

读取之前的扫描结果：

```bash
ncdu -f disk.json
```

## 5. 实用示例

查服务器根目录哪个地方占空间：

```bash
sudo ncdu -x /
```

查日志目录：

```bash
sudo ncdu /var/log
```

查某个用户目录：

```bash
ncdu /home/username
```

注意：`ncdu` 里按 `d` 会真的删除文件，操作前要确认路径和内容。
