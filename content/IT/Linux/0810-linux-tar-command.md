---
title: Linux tar 命令
section: IT
category: Linux
---

# Linux tar 命令

<img src="images/Linux.svg" width="300">

`tar`（Tape Archive）是 Linux 中最常用的文件打包和归档工具，可以将多个文件或目录打包成一个文件，也常与 `gzip`、`bzip2`、`xz` 等压缩工具配合使用。

## 一、基本语法

```bash
tar [选项] [归档文件] [源文件或目录]
```

例如：

```bash
tar -cvf backup.tar /home/user
```

表示将 `/home/user` 目录打包成 `backup.tar`。

---

## 二、常用参数

| 参数               | 说明              |
| ---------------- | --------------- |
| `-c`             | 创建归档文件（create）  |
| `-x`             | 解压归档文件（extract） |
| `-t`             | 查看归档内容（list）    |
| `-v`             | 显示详细过程（verbose） |
| `-f`             | 指定归档文件名（file）   |
| `-z`             | 使用 gzip 压缩或解压   |
| `-j`             | 使用 bzip2 压缩或解压  |
| `-J`             | 使用 xz 压缩或解压     |
| `-C`             | 切换到指定目录操作       |
| `--exclude`      | 排除指定文件或目录       |
| `-p`             | 保留原权限           |
| `--remove-files` | 打包后删除源文件        |

---

## 三、常见使用场景

### 1. 打包文件

```bash
tar -cvf archive.tar file1.txt file2.txt
```

生成：

```text
archive.tar
```

仅打包，不压缩。

---

### 2. 打包目录

```bash
tar -cvf website.tar website/
```

---

### 3. 使用 gzip 压缩

```bash
tar -czvf website.tar.gz website/
```

或者：

```bash
tar -zcvf website.tar.gz website/
```

生成：

```text
website.tar.gz
```

---

### 4. 使用 bzip2 压缩

```bash
tar -cjvf website.tar.bz2 website/
```

---

### 5. 使用 xz 压缩

压缩率更高：

```bash
tar -cJvf website.tar.xz website/
```

---

## 四、查看压缩包内容

### 查看 tar 文件

```bash
tar -tvf archive.tar
```

### 查看 tar.gz 文件

```bash
tar -tzvf archive.tar.gz
```

输出示例：

```text
drwxr-xr-x root/root         0 2025-06-01 website/
-rw-r--r-- root/root      1024 2025-06-01 website/index.html
```

---

## 五、解压文件

### 1. 解压 tar

```bash
tar -xvf archive.tar
```

---

### 2. 解压 tar.gz

```bash
tar -xzvf archive.tar.gz
```

---

### 3. 解压 tar.bz2

```bash
tar -xjvf archive.tar.bz2
```

---

### 4. 解压 tar.xz

```bash
tar -xJvf archive.tar.xz
```

---

## 六、解压到指定目录

```bash
tar -xzvf backup.tar.gz -C /data/restore/
```

说明：

```text
-C /data/restore/
```

表示先进入该目录，再进行解压。

---

## 七、排除文件或目录

例如打包网站时排除日志目录：

```bash
tar -czvf website.tar.gz website/ \
    --exclude='website/logs'
```

排除多个目录：

```bash
tar -czvf backup.tar.gz /data \
    --exclude='/data/logs' \
    --exclude='/data/tmp'
```

---

## 八、只解压指定文件

查看内容：

```bash
tar -tzvf backup.tar.gz
```

解压某个文件：

```bash
tar -xzvf backup.tar.gz etc/nginx/nginx.conf
```

---

## 九、增量备份

备份变化文件：

```bash
tar -czg snapshot.snar -f backup.tar.gz /data
```

再次执行时：

```bash
tar -czg snapshot.snar -f backup2.tar.gz /data
```

只会备份新增或修改的文件。

---

## 十、保留权限和属主信息

系统迁移时常用：

```bash
tar -cpzvf system_backup.tar.gz /
```

恢复：

```bash
tar -xpzvf system_backup.tar.gz
```

其中：

```text
-p 保留原权限
```

---

## 十一、生产环境常用示例

### 备份应用目录

```bash
tar -czvf app_$(date +%F).tar.gz /opt/app
```

生成：

```text
app_2026-06-16.tar.gz
```

---

### 备份数据库导出文件

```bash
mysqldump -uroot -p dbname > db.sql

tar -czvf db_backup.tar.gz db.sql
```

---

### 备份日志

```bash
tar -czvf logs_$(date +%Y%m%d).tar.gz /var/log
```

---

## 十二、速查表

| 操作      | 命令                                           |
| ------- | -------------------------------------------- |
| 打包      | `tar -cvf file.tar dir/`                     |
| 解包      | `tar -xvf file.tar`                          |
| gzip压缩  | `tar -czvf file.tar.gz dir/`                 |
| gzip解压  | `tar -xzvf file.tar.gz`                      |
| bzip2压缩 | `tar -cjvf file.tar.bz2 dir/`                |
| bzip2解压 | `tar -xjvf file.tar.bz2`                     |
| xz压缩    | `tar -cJvf file.tar.xz dir/`                 |
| xz解压    | `tar -xJvf file.tar.xz`                      |
| 查看内容    | `tar -tvf file.tar`                          |
| 解压到指定目录 | `tar -xvf file.tar -C /target`               |
| 排除目录    | `tar --exclude='dir' -czvf file.tar.gz src/` |

对于日常运维工作，最常用的实际上就是下面这几条：

```bash
# 打包并gzip压缩
tar -czvf backup.tar.gz /data

# 解压
tar -xzvf backup.tar.gz

# 查看压缩包内容
tar -tzvf backup.tar.gz

# 解压到指定目录
tar -xzvf backup.tar.gz -C /restore
```

记住规律即可：

* `c` = 创建（Create）
* `x` = 解压（Extract）
* `t` = 查看（Table/List）
* `v` = 显示过程（Verbose）
* `f` = 文件（File）
* `z` = gzip
* `j` = bzip2
* `J` = xz

这样基本就能覆盖 90% 以上的 Linux 日常使用场景。