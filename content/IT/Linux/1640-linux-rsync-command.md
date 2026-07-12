---
title: Linux rsync 命令
section: IT
category: Linux
---

# Linux rsync 命令

<img src="images/Linux.svg" width="300">

`rsync`（Remote Sync）是 Linux 中最常用的文件同步工具之一，它可以在本地目录之间、不同服务器之间高效地同步文件。相比 `cp`、`scp` 等命令，`rsync` 的最大特点是**增量同步**：只传输发生变化的部分，因此速度快、节省带宽。

---

# 一、基本语法

```bash
rsync [选项] 源路径 目标路径
```

例如：

```bash
rsync -av /home/data/ /backup/data/
```

表示把 `/home/data/` 同步到 `/backup/data/`。

---

# 二、常用参数

| 参数                | 作用                                    |
| ----------------- | ------------------------------------- |
| `-a`              | 归档模式（最常用），保留权限、时间、软链接等，相当于 `-rlptgoD` |
| `-v`              | 显示详细过程                                |
| `-z`              | 传输时压缩（网络同步建议开启）                       |
| `-h`              | 人类可读（显示 KB、MB、GB）                     |
| `-P`              | 显示进度，并支持断点续传（=`--progress --partial`） |
| `--delete`        | 删除目标目录中源目录没有的文件                       |
| `--exclude`       | 排除指定文件或目录                             |
| `--include`       | 包含指定文件                                |
| `-n`              | 模拟运行（Dry Run），不真正执行                   |
| `-e ssh`          | 使用 SSH 作为传输协议                         |
| `--bwlimit=10000` | 限制带宽（KB/s）                            |
| `--checksum`      | 根据文件内容判断是否变化，而不是时间和大小                 |

---

# 三、本地同步

假设：

```
/data/source
/data/backup
```

同步：

```bash
rsync -av /data/source/ /data/backup/
```

输出类似：

```
sending incremental file list

file1.txt
dir1/
dir1/test.log

sent 5,102 bytes
received 120 bytes
```

---

## 注意最后有没有 `/`

例如：

```
source/
```

表示同步**目录里面的内容**

```
backup/
```

结果：

```
backup/
    file1
    file2
```

而

```bash
rsync -av source backup/
```

结果会变成

```
backup/
    source/
        file1
        file2
```

这是最容易犯的错误之一。

---

# 四、远程同步（SSH）

## 上传到远程服务器

```bash
rsync -avz -e ssh ./data user@192.168.1.100:/backup/
```

或者

```bash
rsync -avz ./data user@192.168.1.100:/backup/
```

默认就是 SSH。

---

## 从远程下载

```bash
rsync -avz user@192.168.1.100:/backup/data .
```

下载到当前目录。

---

## 指定 SSH 端口

例如 SSH 端口为 2222：

```bash
rsync -avz -e "ssh -p 2222" \
    ./data \
    user@192.168.1.100:/backup/
```

---

# 五、删除目标多余文件

例如：

源目录：

```
a.txt
b.txt
```

目标目录：

```
a.txt
b.txt
c.txt
```

执行：

```bash
rsync -av --delete source/ backup/
```

结果：

```
backup/

a.txt
b.txt
```

`c.txt` 会被删除。

> **建议先使用 `-n`（模拟运行）预览结果，再执行真正同步。**

```bash
rsync -avn --delete source/ backup/
```

---

# 六、排除文件

例如：

```bash
rsync -av \
    --exclude "*.log" \
    --exclude "temp/" \
    source/ backup/
```

不会同步：

```
*.log
temp/
```

---

也可以写排除文件：

```
exclude.txt

*.log
*.tmp
cache/
```

然后：

```bash
rsync -av --exclude-from=exclude.txt source backup
```

---

# 七、显示同步进度

```bash
rsync -avP source/ backup/
```

输出：

```
12,584,233  55%  25.12MB/s 0:00:02
```

`-P` 是最常用参数之一。

---

# 八、断点续传

如果网络中断：

```bash
rsync -avP bigfile.iso user@host:/backup/
```

再次执行：

```bash
rsync -avP bigfile.iso user@host:/backup/
```

会从中断处继续传输。

---

# 九、模拟执行

真正同步之前建议：

```bash
rsync -avhn source/ backup/
```

输出：

```
sending incremental file list

test.txt
dir/
```

不会真正复制文件。

---

# 十、镜像备份

保持两个目录完全一致：

```bash
rsync -av --delete source/ backup/
```

这是很多备份脚本的标准写法。

---

# 十一、限制带宽

例如限制 20 MB/s：

```bash
rsync -avz --bwlimit=20480 source user@host:/backup/
```

单位是 **KB/s**。

---

# 十二、典型应用场景

## 1. 每天备份网站

```bash
rsync -av --delete /var/www/ /backup/www/
```

---

## 2. 同步日志（排除缓存）

```bash
rsync -av \
    --exclude cache \
    /var/log \
    backup/
```

---

## 3. 同步到远程服务器

```bash
rsync -avzP \
    /data \
    user@server:/backup/
```

---

## 4. 下载远程备份

```bash
rsync -avzP \
    user@server:/backup/db.sql \
    ./
```

---

## 5. 使用 SSH 密钥免密同步

```bash
rsync -avz -e ssh \
    /data \
    user@host:/backup/
```

前提是在两台机器之间配置好 SSH 公钥认证，这样可以方便地通过 `cron` 或 `systemd timer` 实现自动同步。

---

# 十三、`rsync` 与其他命令的比较

| 命令      | 是否增量同步 | 是否支持断点续传 | 是否保留权限 | 是否适合远程 |
| ------- | ------ | -------- | ------ | ------ |
| `cp`    | ❌      | ❌        | 部分支持   | ❌      |
| `scp`   | ❌      | ❌        | 基本支持   | ✅      |
| `tar`   | ❌      | ❌        | ✅      | ❌      |
| `rsync` | ✅      | ✅        | ✅      | ✅      |

---

# 十四、最佳实践

对于本地或远程同步，以下命令组合最常用：

```bash
rsync -avhP source/ destination/
```

如果需要让目标目录与源目录保持完全一致：

```bash
rsync -avhP --delete source/ destination/
```

在生产环境中，建议先进行模拟执行，确认将要发生的变更：

```bash
rsync -avhn --delete source/ destination/
```

确认无误后再去掉 `-n` 参数执行正式同步。这样可以有效避免因路径（尤其是源路径末尾是否带 `/`）或 `--delete` 参数使用不当而造成的数据误删。