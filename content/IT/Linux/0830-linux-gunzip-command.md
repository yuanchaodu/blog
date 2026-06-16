---
title: Linux gunzip 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKkT
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/83'
---

# Linux gunzip 命令

<img src="images/Linux.svg" width="300">

`gunzip` 是 Linux/Unix 系统中用于**解压 `.gz` 压缩文件**的命令，它是 GNU gzip 工具集的一部分。

## 一、基本语法

```bash
gunzip [选项] 文件.gz
```

例如：

```bash
gunzip test.txt.gz
```

执行后：

```text
test.txt.gz  →  test.txt
```

默认会删除原来的 `.gz` 文件，只保留解压后的文件。

---

## 二、常用用法

### 1. 解压单个文件

```bash
gunzip file.gz
```

等价于：

```bash
gzip -d file.gz
```

---

### 2. 解压多个文件

```bash
gunzip *.gz
```

例如：

```bash
gunzip log1.gz log2.gz log3.gz
```

---

### 3. 保留原压缩文件

`gunzip` 本身没有直接保留原文件的参数（部分新版 gzip 支持 `-k`）。

```bash
gzip -dk file.gz
```

或者：

```bash
gunzip -c file.gz > file
```

说明：

* `-c`：输出到标准输出
* 原 `.gz` 文件不会删除

示例：

```bash
gunzip -c test.txt.gz > test.txt
```

结果：

```text
test.txt.gz （保留）
test.txt    （新生成）
```

---

### 4. 查看压缩文件内容（不解压）

```bash
zcat file.gz
```

例如：

```bash
zcat access.log.gz
```

配合分页查看：

```bash
zcat access.log.gz | less
```

---

### 5. 查看压缩文件信息

```bash
gzip -l file.gz
```

例如：

```bash
gzip -l backup.tar.gz
```

输出类似：

```text
compressed  uncompressed  ratio
12345       67890         81.8%
```

---

## 三、解压 tar.gz 文件

很多人会误用 `gunzip` 解压 `.tar.gz` 文件。

例如：

```text
backup.tar.gz
```

实际上包含两层：

```text
backup.tar.gz
    ↓ gunzip
backup.tar
    ↓ tar
文件目录
```

推荐直接使用：

```bash
tar -zxvf backup.tar.gz
```

参数说明：

* `z`：gzip解压
* `x`：解包
* `v`：显示过程
* `f`：指定文件

---

## 四、常用选项

| 选项   | 说明               |
| ---- | ---------------- |
| `-c` | 输出到标准输出          |
| `-f` | 强制解压             |
| `-k` | 保留原压缩文件（新版 gzip） |
| `-l` | 查看压缩文件信息         |
| `-r` | 递归处理目录           |
| `-t` | 测试压缩文件完整性        |
| `-v` | 显示详细信息           |

示例：

```bash
gunzip -v test.gz
```

输出：

```text
test.gz: 78.3% -- replaced with test
```

---

## 五、完整性检查

检查 `.gz` 文件是否损坏：

```bash
gunzip -t file.gz
```

或者：

```bash
gzip -t file.gz
```

正常无输出：

```bash
echo $?
```

返回：

```text
0
```

表示文件完整。

---

## 六、实际运维场景

### 解压日志文件

```bash
gunzip access.log.20250601.gz
```

---

### 查看压缩日志而不解压

```bash
zcat access.log.gz | grep ERROR
```

---

### 批量解压目录下所有日志

```bash
find /data/logs -name "*.gz" -exec gunzip {} \;
```

---

## 七、gzip、gunzip、zcat关系

| 命令       | 功能       |
| -------- | -------- |
| `gzip`   | 压缩文件     |
| `gunzip` | 解压文件     |
| `zcat`   | 查看压缩文件内容 |
| `gzexe`  | 压缩可执行文件  |

例如：

```bash
gzip test.txt
```

生成：

```text
test.txt.gz
```

再执行：

```bash
gunzip test.txt.gz
```

恢复：

```text
test.txt
```

因此，`gunzip` 实际上就是：

```bash
gzip -d
```

的快捷写法。
