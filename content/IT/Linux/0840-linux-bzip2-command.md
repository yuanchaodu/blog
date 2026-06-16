---
title: Linux bzip2 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKkz
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/84'
---

# Linux bzip2 命令

<img src="images/Linux.svg" width="300">

`bzip2` 是 Linux 中常用的文件压缩工具，采用 Burrows-Wheeler 算法，压缩率通常比 `gzip` 高，但压缩速度相对较慢。它主要用于压缩单个文件，不会像 `tar` 一样打包目录。

## 一、基本语法

```bash
bzip2 [选项] 文件名
```

压缩后会生成：

```text
file.txt → file.txt.bz2
```

默认情况下，原文件会被删除，只保留压缩文件。

---

## 二、常用操作

### 1. 压缩文件

```bash
bzip2 test.txt
```

结果：

```text
test.txt.bz2
```

原来的 `test.txt` 被删除。

---

### 2. 保留原文件

```bash
bzip2 -k test.txt
```

或

```bash
bzip2 --keep test.txt
```

结果：

```text
test.txt
test.txt.bz2
```

---

### 3. 解压文件

```bash
bunzip2 test.txt.bz2
```

或

```bash
bzip2 -d test.txt.bz2
```

结果：

```text
test.txt
```

默认删除 `.bz2` 文件。

---

### 4. 解压并保留压缩包

```bash
bzip2 -dk test.txt.bz2
```

---

## 三、查看压缩文件内容

如果是文本文件：

```bash
bzcat test.txt.bz2
```

输出内容到终端：

```bash
bzcat test.txt.bz2 | less
```

---

## 四、测试压缩包完整性

```bash
bzip2 -t test.txt.bz2
```

如果正常：

```text
ok
```

如果损坏会报错。

---

## 五、设置压缩级别

压缩级别范围：

```text
-1  压缩最快
-9  压缩率最高（默认）
```

例如：

```bash
bzip2 -1 logfile.log
```

最高压缩率：

```bash
bzip2 -9 logfile.log
```

---

## 六、批量压缩

压缩当前目录所有日志文件：

```bash
bzip2 *.log
```

---

## 七、与 tar 配合使用

实际生产环境中，`bzip2` 通常与 `tar` 一起使用压缩目录。

### 打包并压缩目录

```bash
tar -cjf backup.tar.bz2 /data
```

参数说明：

```text
-c  创建归档
-j  使用 bzip2 压缩
-f  指定文件名
```

---

### 解压 tar.bz2 文件

```bash
tar -xjf backup.tar.bz2
```

---

### 查看归档内容

```bash
tar -tjf backup.tar.bz2
```

---

## 八、常用选项汇总

| 选项          | 说明      |
| ----------- | ------- |
| `-d`        | 解压缩     |
| `-k`        | 保留原文件   |
| `-f`        | 强制覆盖    |
| `-t`        | 测试文件完整性 |
| `-1` ~ `-9` | 压缩级别    |
| `-v`        | 显示详细过程  |
| `-c`        | 输出到标准输出 |
| `-q`        | 安静模式    |

---

## 九、实际运维示例

### 压缩大型日志

```bash
bzip2 access.log
```

生成：

```text
access.log.bz2
```

---

### 查看压缩日志

```bash
bzcat access.log.bz2 | grep ERROR
```

---

### 压缩数据库备份文件

```bash
mysqldump -uroot -p dbname > db.sql
bzip2 db.sql
```

得到：

```text
db.sql.bz2
```

---

## 十、与 gzip、xz 对比

| 工具    | 扩展名  | 压缩率 | 压缩速度 | 解压速度 |
| ----- | ---- | --- | ---- | ---- |
| gzip  | .gz  | 中   | 快    | 快    |
| bzip2 | .bz2 | 较高  | 较慢   | 中    |
| xz    | .xz  | 最高  | 最慢   | 较慢   |

一般建议：

* 日常日志压缩：`gzip`
* 长期归档：`bzip2`
* 追求最高压缩率：`xz`

例如：

```bash
tar -cJf archive.tar.xz data/
```

压缩率通常会高于：

````bash
tar -cjf archive.tar.bz2 data/
```。

对于现代 Linux 系统，很多场景已经更多使用 `xz` 或 `zstd`，但 `bzip2` 仍广泛用于历史归档文件和一些软件发行包中。
````
