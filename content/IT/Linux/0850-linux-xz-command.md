---
title: Linux xz 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKlj
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/85'
---

# Linux xz 命令

<img src="images/Linux.svg" width="300">

`xz` 是 Linux 中常用的文件压缩工具，采用 **LZMA2** 压缩算法，特点是**压缩率高**，生成的文件扩展名通常为 `.xz`。

## 一、基本语法

```bash
xz [选项] 文件名
```

例如：

```bash
xz test.txt
```

执行后：

```text
test.txt
↓
test.txt.xz
```

原文件默认会被删除，只保留压缩后的文件。

---

## 二、常用压缩命令

### 1. 压缩文件

```bash
xz file.txt
```

生成：

```text
file.txt.xz
```

---

### 2. 保留原文件

```bash
xz -k file.txt
```

或

```bash
xz --keep file.txt
```

结果：

```text
file.txt
file.txt.xz
```

两个文件都会保留。

---

### 3. 指定压缩级别

压缩级别范围：

```text
-0 ~ -9
```

默认：

```text
-6
```

例如：

```bash
xz -9 file.txt
```

特点：

| 级别 | 压缩速度 | 压缩率 |
| -- | ---- | --- |
| -1 | 快    | 较低  |
| -6 | 默认   | 平衡  |
| -9 | 慢    | 最高  |

---

### 4. 使用多线程压缩

查看 CPU 核数：

```bash
nproc
```

使用全部核心：

```bash
xz -T0 file.txt
```

指定线程数：

```bash
xz -T4 file.txt
```

说明：

```text
-T0  自动使用所有CPU核心
-T4  使用4个线程
```

对于大文件压缩效果明显。

---

## 三、解压命令

### 1. 解压

```bash
xz -d file.txt.xz
```

或

```bash
unxz file.txt.xz
```

结果：

```text
file.txt
```

---

### 2. 解压并保留压缩包

```bash
xz -dk file.txt.xz
```

结果：

```text
file.txt
file.txt.xz
```

---

### 3. 输出到标准输出

```bash
xz -dc file.txt.xz
```

例如：

```bash
xz -dc file.txt.xz | less
```

无需解压到磁盘即可查看内容。

---

## 四、查看压缩文件信息

### 查看文件大小

```bash
xz -l file.txt.xz
```

示例输出：

```text
Strms  Blocks   Compressed Uncompressed  Ratio  Check   Filename
1      1        12.5 MiB   48.3 MiB      0.259  CRC64   file.txt.xz
```

---

### 详细信息

```bash
xz -lv file.txt.xz
```

---

## 五、与 tar 结合使用

实际生产环境中最常见的是：

### 打包并压缩

```bash
tar -cJf backup.tar.xz /data
```

参数说明：

```text
-c  创建归档
-J  使用xz压缩
-f  指定文件名
```

生成：

```text
backup.tar.xz
```

---

### 解压 tar.xz

```bash
tar -xJf backup.tar.xz
```

或者：

```bash
tar -xf backup.tar.xz
```

现代 Linux 通常能自动识别压缩格式。

---

## 六、测试压缩包完整性

```bash
xz -t file.txt.xz
```

如果文件正常：

```text
无输出
```

查看返回值：

```bash
echo $?
```

```text
0
```

表示文件完整。

---

## 七、查看 xz 版本

```bash
xz --version
```

例如：

```text
xz (XZ Utils) 5.6.2
liblzma 5.6.2
```

---

## 八、实际工作中的典型用法

### 压缩日志

```bash
xz -T0 app.log
```

---

### 批量压缩

```bash
find /logs -name "*.log" -exec xz {} \;
```

---

### 查看压缩日志内容

```bash
xzcat app.log.xz
```

相当于：

```bash
xz -dc app.log.xz
```

配合：

```bash
xzcat app.log.xz | grep ERROR
```

---

## 九、与 gzip、bzip2 对比

| 工具    | 扩展名  | 压缩速度 | 压缩率 |
| ----- | ---- | ---- | --- |
| gzip  | .gz  | 快    | 一般  |
| bzip2 | .bz2 | 较慢   | 较高  |
| xz    | .xz  | 慢    | 最高  |
| zstd  | .zst | 很快   | 高   |

目前在 Linux 服务器场景中：

* 日常日志压缩：优先 `gzip` 或 `zstd`
* 长期归档存储：优先 `xz`
* 大规模备份：推荐 `tar + xz`
* 高性能场景：推荐 `zstd`

例如化工企业的历史生产数据归档、日志长期保存、系统备份文件保存，常见做法是：

```bash
tar -cJf backup_$(date +%F).tar.xz /data
```

这样可以在保证较高压缩率的同时，减少存储空间占用。
