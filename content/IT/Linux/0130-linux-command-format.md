---
title: Linux 命令格式
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnB1z
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/13'
---

# Linux 命令格式

<img src="images/Linux.svg" width="300">

Linux 命令一般格式如下：

```bash
命令 [选项] [参数]
```

含义：

```bash
ls -l /home
```

其中：

* `ls`：命令，表示列出文件
* `-l`：选项，表示用详细格式显示
* `/home`：参数，表示要查看的目录

常见格式示例：

```bash
命令
命令 -选项
命令 --长选项
命令 参数
命令 -选项 参数
```

例子：

```bash
pwd
ls -a
ls --all
cd /etc
cp a.txt b.txt
rm -r test
```

补充说明：

短选项通常用一个横杠：

```bash
ls -l
```

长选项通常用两个横杠：

```bash
ls --help
```

多个短选项可以合并：

```bash
ls -l -a
ls -la
```
