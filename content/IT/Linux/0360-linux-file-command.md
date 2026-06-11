---
title: Linux file 命令
section: IT
category: Linux
---

# Linux file 命令

<img src="images/Linux.svg" width="300">

`file` 是 Linux 中一个非常实用的命令，用于**识别文件类型**。它不像 Windows 那样主要依赖文件扩展名，而是通过分析文件内容中的特征（Magic Number）来判断文件实际类型。

## 基本语法

```bash
file [选项] 文件名
```

例如：

```bash
file test.txt
```

输出：

```text
test.txt: ASCII text
```

表示这是一个文本文件。

---

## 常见用法

### 1. 查看文件类型

```bash
file example
```

输出示例：

```text
example: ELF 64-bit LSB executable, x86-64
```

说明这是一个 Linux 可执行程序。

---

### 2. 查看多个文件

```bash
file *
```

输出：

```text
a.txt:      ASCII text
b.jpg:      JPEG image data
script.sh:  Bourne-Again shell script
```

---

### 3. 查看符号链接指向的文件

默认情况下：

```bash
file linkfile
```

输出：

```text
linkfile: symbolic link to target.txt
```

跟踪链接：

```bash
file -L linkfile
```

输出：

```text
linkfile: ASCII text
```

---

### 4. 简洁输出

```bash
file -b test.txt
```

输出：

```text
ASCII text
```

不显示文件名。

---

### 5. 查看 MIME 类型

对于 Web 服务、邮件系统等场景很有用：

```bash
file --mime-type test.pdf
```

输出：

```text
test.pdf: application/pdf
```

或者：

```bash
file -i test.pdf
```

输出：

```text
test.pdf: application/pdf; charset=binary
```

---

## 常见识别结果

### 文本文件

```bash
file readme.txt
```

输出：

```text
readme.txt: UTF-8 Unicode text
```

---

### Shell脚本

```bash
file backup.sh
```

输出：

```text
backup.sh: Bourne-Again shell script, UTF-8 text executable
```

---

### 图片文件

```bash
file image.png
```

输出：

```text
image.png: PNG image data, 1920 x 1080
```

---

### 压缩文件

```bash
file archive.tar.gz
```

输出：

```text
archive.tar.gz: gzip compressed data
```

---

### ZIP文件

```bash
file package.zip
```

输出：

```text
package.zip: Zip archive data
```

---

### ELF可执行文件

```bash
file /bin/ls
```

输出：

```text
/bin/ls: ELF 64-bit LSB pie executable, x86-64
```

---

## 实际运维中的典型用途

### 判断文件真实类型

有些文件扩展名被修改过：

```bash
file suspicious.txt
```

输出：

```text
suspicious.txt: JPEG image data
```

说明虽然扩展名是 `.txt`，实际却是图片。

---

### 检查日志文件编码

```bash
file log.txt
```

输出：

```text
log.txt: UTF-8 Unicode text
```

或者：

```text
log.txt: ISO-8859 text
```

便于判断是否需要转码。

---

### 批量检查目录文件类型

```bash
find . -type f | xargs file
```

或者：

```bash
find . -type f -exec file {} \;
```

---

## 工作原理

`file` 命令依赖系统中的 Magic 数据库：

```bash
/usr/share/misc/magic
```

或

```bash
/usr/share/file/magic
```

通过检查文件头部特征来识别类型，例如：

| 文件类型 | 文件头(Magic Number) |
| ---- | ----------------- |
| PNG  | 89 50 4E 47       |
| JPEG | FF D8 FF          |
| PDF  | %PDF              |
| ZIP  | PK                |
| ELF  | 7F 45 4C 46       |

因此：

```bash
mv test.jpg test.txt
file test.txt
```

仍然会显示：

```text
test.txt: JPEG image data
```

因为 `file` 看的是内容而不是扩展名。

---

## 常用选项汇总

| 选项            | 说明          |
| ------------- | ----------- |
| `-b`          | 不显示文件名      |
| `-L`          | 跟踪符号链接      |
| `-i`          | 显示 MIME 类型  |
| `--mime-type` | 仅显示 MIME 类型 |
| `-z`          | 分析压缩文件内容    |
| `-k`          | 显示多个匹配结果    |
| `-f 文件列表`     | 从文件读取待检测文件名 |

例如：

```bash
file -z backup.tar.gz
```

可能输出：

```text
backup.tar.gz: gzip compressed data, was "backup.tar", ...
```

甚至进一步识别出内部是 tar 包。

---

对于系统运维和故障排查，`file` 是一个经常与 `ls`、`stat`、`hexdump`、`strings`、`readelf` 配合使用的基础工具。遇到“这个文件到底是什么”“为什么打不开”“扩展名是否被篡改”等问题时，通常第一步就是执行：

```bash
file 文件名
```