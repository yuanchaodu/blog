---
title: Linux tree 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnCCm
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/24'
---

# Linux tree 命令

<img src="images/Linux.svg" width="300">

`tree` 是 Linux 中一个非常实用的目录查看工具，可以用树状结构显示目录和文件层级关系，比 `ls -R` 更直观。

## 安装 tree

不同发行版安装方法：

### CentOS / RHEL

```bash
yum install tree
```

或

```bash
dnf install tree
```

### Ubuntu / Debian

```bash
sudo apt install tree
```

### openSUSE

```bash
sudo zypper install tree
```

---

## 基本用法

### 查看当前目录

```bash
tree
```

例如：

```text
.
├── file1.txt
├── file2.txt
├── dir1
│   ├── a.txt
│   └── b.txt
└── dir2
    └── c.txt

2 directories, 5 files
```

---

### 查看指定目录

```bash
tree /opt
```

---

## 常用参数

### 1. 只显示目录

```bash
tree -d
```

示例：

```text
.
├── backup
├── logs
└── scripts
```

---

### 2. 限制显示层级

只显示两层：

```bash
tree -L 2
```

示例：

```text
.
├── dir1
│   ├── a
│   └── b
└── dir2
    └── c
```

---

### 3. 显示隐藏文件

```bash
tree -a
```

会显示：

```text
.
├── .bashrc
├── .vimrc
└── test.txt
```

---

### 4. 显示文件大小

```bash
tree -h
```

示例：

```text
.
├── [4.0K] file1.txt
├── [2.1M] data.zip
└── [512 ] test.log
```

---

### 5. 显示权限

```bash
tree -p
```

示例：

```text
.
├── [-rw-r--r--] file1.txt
└── [drwxr-xr-x] scripts
```

---

### 6. 显示文件属主

```bash
tree -u
```

显示用户：

```bash
tree -ug
```

显示用户和组：

```text
.
├── [root root] file1
└── [nginx nginx] web
```

---

### 7. 按时间排序

```bash
tree -t
```

最新修改的文件优先显示。

---

### 8. 统计目录大小

```bash
tree --du
```

示例：

```text
[  25M]  .
├── [  10M] logs
├── [  12M] backup
└── [   3M] scripts
```

这在排查磁盘空间占用时非常有用。

---

### 9. 排除目录或文件

排除 `node_modules`：

```bash
tree -I node_modules
```

排除多个目录：

```bash
tree -I "node_modules|.git|dist"
```

---

### 10. 只显示匹配文件

查看所有日志文件：

```bash
tree -P "*.log"
```

示例：

```text
.
├── app.log
└── logs
    └── error.log
```

---

## 导出结果

### 导出到文本文件

```bash
tree > tree.txt
```

### 导出 HTML

```bash
tree -H . -o tree.html
```

生成可在浏览器查看的目录树。

---

## 实际工作中的常用场景

### 查看应用部署结构

```bash
tree -L 2 /opt/app
```

### 查看网站目录

```bash
tree -L 3 /var/www/html
```

### 查看日志目录

```bash
tree -h /var/log
```

### 排查磁盘空间

```bash
tree -h --du /data
```

### 查看项目结构（排除 Git 和依赖）

```bash
tree -I ".git|node_modules|venv"
```

例如：

```bash
tree -L 3 -I ".git|node_modules"
```

输出：

```text
project
├── src
│   ├── main.py
│   └── utils.py
├── config
│   └── app.yaml
├── logs
└── README.md
```

---

## 与 find 命令结合

统计目录中文件数量：

```bash
tree | tail -1
```

查看某类文件：

```bash
tree -P "*.conf"
```

如果系统没有安装 `tree`，也可以用：

```bash
find . | sed -e 's/[^-][^\/]*\//│   /g' -e 's/│   \([^│]\)/├── \1/'
```

模拟简单的树状显示，不过可读性和功能都不如 `tree`。

对于日常运维、系统排障和项目结构查看，最常用的组合通常是：

```bash
tree -L 3
tree -h
tree -d
tree -I ".git|node_modules"
tree --du -h
```

这几个参数基本覆盖了 90% 的使用场景。
