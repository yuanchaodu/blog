---
title: Linux gzip 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKa8
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/82'
---

# Linux gzip 命令

<img src="images/Linux.svg" width="300">

`gzip` 是 Linux 中常用的文件压缩工具，主要用于压缩和解压缩文件，压缩后的文件扩展名通常为 `.gz`。

## 一、基本语法

```bash
gzip [选项] 文件名
```

压缩后，原文件默认会被删除，只保留压缩文件。

例如：

```bash
gzip test.txt
```

执行后：

```text
test.txt
```

变成：

```text
test.txt.gz
```

---

## 二、常用命令

### 1. 压缩文件

```bash
gzip file.txt
```

生成：

```bash
file.txt.gz
```

---

### 2. 保留原文件

默认情况下 gzip 会删除原文件，如果希望保留原文件：

```bash
gzip -c file.txt > file.txt.gz
```

或者：

```bash
cat file.txt | gzip > file.txt.gz
```

---

### 3. 解压文件

```bash
gzip -d file.txt.gz
```

等同于：

```bash
gunzip file.txt.gz
```

解压后得到：

```bash
file.txt
```

---

### 4. 查看压缩文件内容

无需解压即可查看：

```bash
zcat file.txt.gz
```

配合 grep：

```bash
zcat file.txt.gz | grep ERROR
```

查看前几行：

```bash
zcat file.txt.gz | head
```

---

### 5. 递归压缩目录中的文件

gzip 本身不能直接压缩目录：

错误示例：

```bash
gzip mydir
```

会提示：

```text
gzip: mydir is a directory -- ignored
```

通常先使用 tar 打包：

```bash
tar -czvf mydir.tar.gz mydir
```

参数说明：

| 参数 | 含义         |
| -- | ---------- |
| c  | 创建归档       |
| z  | 使用 gzip 压缩 |
| v  | 显示过程       |
| f  | 指定文件名      |

生成：

```bash
mydir.tar.gz
```

---

### 6. 解压 tar.gz 文件

```bash
tar -xzvf mydir.tar.gz
```

---

## 三、压缩级别

gzip 支持 1～9 级压缩：

```bash
gzip -1 file.txt
```

最快压缩，压缩率较低。

```bash
gzip -9 file.txt
```

最高压缩率，但耗时较长。

默认：

```bash
gzip -6 file.txt
```

---

## 四、查看压缩文件信息

```bash
gzip -l file.txt.gz
```

输出示例：

```text
compressed        uncompressed  ratio uncompressed_name
      1024                4096  75.0% file.txt
```

字段说明：

| 字段                | 含义    |
| ----------------- | ----- |
| compressed        | 压缩后大小 |
| uncompressed      | 原始大小  |
| ratio             | 压缩率   |
| uncompressed_name | 原文件名  |

---

## 五、批量压缩

压缩当前目录所有日志文件：

```bash
gzip *.log
```

---

压缩所有 txt 文件：

```bash
find . -name "*.txt" -exec gzip {} \;
```

---

## 六、批量解压

解压当前目录所有 gz 文件：

```bash
gunzip *.gz
```

或者：

```bash
gzip -d *.gz
```

---

## 七、生产环境常见用法

### 压缩历史日志

```bash
gzip app.log
```

生成：

```bash
app.log.gz
```

节省磁盘空间。

---

### 查看压缩日志中的错误

```bash
zgrep ERROR app.log.gz
```

相当于：

```bash
zcat app.log.gz | grep ERROR
```

---

### 查看压缩日志最后几行

```bash
zcat app.log.gz | tail
```

或者：

```bash
zless app.log.gz
```

---

### 打包备份目录

```bash
tar -czvf backup_$(date +%F).tar.gz /data
```

生成类似：

```text
backup_2026-06-16.tar.gz
```

这是 Linux 服务器备份中最常见的做法之一。

---

## 八、与 zip 的区别

| 对比项       | gzip | zip     |
| --------- | ---- | ------- |
| 压缩对象      | 单个文件 | 多个文件和目录 |
| Linux原生支持 | 很好   | 好       |
| 压缩率       | 较高   | 一般      |
| Windows支持 | 需工具  | 原生支持    |
| 常见格式      | .gz  | .zip    |

在 Linux 服务器运维中：

* 压缩单个日志文件：`gzip`
* 压缩整个目录：`tar + gzip（.tar.gz）`
* 与 Windows 用户交换文件：`zip`

这是最常见的使用习惯。
