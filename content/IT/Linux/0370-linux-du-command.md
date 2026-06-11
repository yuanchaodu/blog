---
title: Linux du 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDhB
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/37'
---

# Linux du 命令

<img src="images/Linux.svg" width="300">

`du`（Disk Usage）是 Linux 中用于查看**目录或文件占用磁盘空间**的常用命令。

## 基本语法

```bash
du [选项] [文件或目录]
```

例如：

```bash
du /home
```

显示 `/home` 目录下所有文件和子目录占用的空间。

---

## 常用参数

### 1. 查看当前目录各文件夹大小

```bash
du
```

输出类似：

```text
4       ./test
8       ./logs
16      .
```

默认单位是 KB。

---

### 2. 以人类可读格式显示

```bash
du -h
```

输出：

```text
4.0K    ./test
8.0K    ./logs
16K     .
```

常见单位：

* K：KB
* M：MB
* G：GB
* T：TB

---

### 3. 只显示目录总大小

```bash
du -sh /data
```

示例：

```text
12G     /data
```

参数说明：

* `-s`：汇总（summary）
* `-h`：人类可读格式

这是最常用的用法。

---

### 4. 查看指定目录下一级子目录大小

```bash
du -h --max-depth=1
```

或：

```bash
du -h -d 1
```

输出：

```text
2.1G    ./backup
5.4G    ./logs
800M    ./data
8.3G    .
```

用于快速找出哪个目录最占空间。

---

### 5. 查看并按大小排序

找出当前目录下最大的文件夹：

```bash
du -h --max-depth=1 | sort -hr
```

结果：

```text
8.3G    .
5.4G    ./logs
2.1G    ./backup
800M    ./data
```

参数说明：

* `sort -h`：按容量排序
* `sort -r`：倒序（从大到小）

---

### 6. 查看指定文件大小

```bash
du -h test.log
```

输出：

```text
128M    test.log
```

---

### 7. 查看多个目录大小

```bash
du -sh /var/log /home /data
```

输出：

```text
3.2G    /var/log
15G     /home
120G    /data
```

---

## 查找最占空间的目录

### 查看根目录下各目录大小

```bash
du -h --max-depth=1 / | sort -hr
```

示例：

```text
120G    /
50G     /data
30G     /home
20G     /var
5G      /usr
```

---

### 查看 /var 下最大的目录

```bash
du -h --max-depth=1 /var | sort -hr
```

---

## du 与 df 的区别

很多人容易混淆：

| 命令   | 作用             |
| ---- | -------------- |
| `du` | 查看目录或文件占用了多少空间 |
| `df` | 查看文件系统剩余多少空间   |

例如：

查看磁盘整体使用情况：

```bash
df -h
```

查看某目录占用情况：

```bash
du -sh /data
```

---

## 生产环境常用组合

### 查看当前目录下最大的 10 个目录

```bash
du -h --max-depth=1 | sort -hr | head -10
```

### 查找根目录下占用空间最大的目录

```bash
du -xh --max-depth=1 / | sort -hr
```

参数：

* `-x`：不跨文件系统（避免统计挂载的 NFS、NAS、Docker 卷等）

### 查找大于 1GB 的文件

配合 `find`：

```bash
find / -type f -size +1G
```

---

## 实战排查磁盘空间不足

当服务器磁盘告警时，通常按下面步骤排查：

```bash
# 1. 查看磁盘使用率
df -h

# 2. 找出根目录下最大的目录
du -xh --max-depth=1 / | sort -hr

# 3. 继续进入大目录
du -xh --max-depth=1 /var | sort -hr

# 4. 继续深入定位
du -xh --max-depth=1 /var/log | sort -hr
```

这样逐层缩小范围，很快就能找到占用空间异常的目录或日志文件。

对于 Linux 运维来说，最常用的三个命令是：

```bash
du -sh *
du -h --max-depth=1 | sort -hr
df -h
```

基本可以解决 80% 以上的磁盘空间排查问题。
