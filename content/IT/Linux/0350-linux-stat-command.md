---
title: Linux stat 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDd4
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/35'
---

# Linux stat 命令

<img src="images/Linux.svg" width="300">

`stat` 是 Linux 中用于**查看文件或文件系统详细状态信息**的命令。相比 `ls -l`，`stat` 能显示更完整的信息，例如文件大小、权限、inode、访问时间、修改时间、创建时间等。

---

# 一、基本语法

```bash
stat [选项] 文件名
```

例如：

```bash
stat test.txt
```

输出示例：

```text
  File: test.txt
  Size: 1024       Blocks: 8          IO Block: 4096 regular file
Device: 8,1        Inode: 123456      Links: 1
Access: (0644/-rw-r--r--)  Uid: (1000/nick)   Gid: (1000/nick)
Access: 2026-06-11 10:00:00.000000000 +0800
Modify: 2026-06-10 15:30:00.000000000 +0800
Change: 2026-06-10 15:35:00.000000000 +0800
 Birth: 2026-06-10 15:20:00.000000000 +0800
```

---

# 二、输出内容解释

## File

```text
File: test.txt
```

文件名。

---

## Size

```text
Size: 1024
```

文件大小（字节）。

---

## Blocks

```text
Blocks: 8
```

文件占用的数据块数量。

---

## IO Block

```text
IO Block: 4096
```

文件系统的块大小。

---

## Device

```text
Device: 8,1
```

所在设备编号。

---

## Inode

```text
Inode: 123456
```

inode 编号。

Linux 通过 inode 管理文件元数据。

---

## Links

```text
Links: 1
```

硬链接数量。

---

## Access

```text
Access: (0644/-rw-r--r--)
```

权限信息：

| 数字 | 权限  |
| -- | --- |
| 6  | rw- |
| 4  | r-- |
| 4  | r-- |

即：

```text
-rw-r--r--
```

---

## Uid/Gid

```text
Uid: (1000/nick)
Gid: (1000/nick)
```

文件所属用户和组。

---

# 三、三个重要时间

Linux 文件通常记录三个时间。

## 1. Access Time（atime）

访问时间。

```bash
cat test.txt
```

读取文件后会更新。

---

## 2. Modify Time（mtime）

内容修改时间。

```bash
echo hello >> test.txt
```

会更新 mtime。

---

## 3. Change Time（ctime）

元数据变更时间。

例如：

```bash
chmod 755 test.txt
```

即使内容没变，也会更新 ctime。

---

### 三种时间关系

```text
atime → 文件被读取
mtime → 文件内容被修改
ctime → 文件属性被修改
```

---

# 四、查看目录信息

```bash
stat /etc
```

输出类似：

```text
File: /etc
Size: 4096
FileType: directory
```

说明 `/etc` 是目录。

---

# 五、自定义输出格式

使用 `-c`（或 `--format`）。

## 查看文件大小

```bash
stat -c %s test.txt
```

输出：

```text
1024
```

---

## 查看权限

```bash
stat -c %a test.txt
```

输出：

```text
644
```

---

## 查看 inode

```bash
stat -c %i test.txt
```

输出：

```text
123456
```

---

## 查看所有者

```bash
stat -c %U test.txt
```

输出：

```text
nick
```

---

## 查看修改时间

```bash
stat -c %y test.txt
```

输出：

```text
2026-06-10 15:30:00.000000000 +0800
```

---

# 六、常用格式符

| 格式符  | 含义       |
| ---- | -------- |
| `%n` | 文件名      |
| `%s` | 文件大小（字节） |
| `%a` | 权限（数字）   |
| `%A` | 权限（字符）   |
| `%U` | 用户名      |
| `%G` | 组名       |
| `%i` | inode    |
| `%h` | 硬链接数     |
| `%x` | 访问时间     |
| `%y` | 修改时间     |
| `%z` | 状态变更时间   |
| `%F` | 文件类型     |

例如：

```bash
stat -c "%n %s %A %U %G" test.txt
```

输出：

```text
test.txt 1024 -rw-r--r-- nick nick
```

---

# 七、查看文件系统信息

使用 `-f`。

```bash
stat -f /
```

输出示例：

```text
File: "/"
    ID: 0        Namelen: 255
Type: ext2/ext3
Block size: 4096
Blocks: Total: 5242880
        Free: 3200000
```

用于查看：

* 文件系统类型
* 总块数
* 可用空间
* 最大文件名长度

---

# 八、与 ls -l 的区别

## ls -l

```bash
ls -l test.txt
```

输出：

```text
-rw-r--r-- 1 nick nick 1024 Jun 10 15:30 test.txt
```

显示的信息较少。

---

## stat

```bash
stat test.txt
```

除了权限和大小外，还能看到：

* inode
* 三种时间
* 文件系统块信息
* UID/GID
* 创建时间（部分文件系统支持）

---

# 九、运维常见场景

### 查看配置文件最近是否被修改

```bash
stat /etc/nginx/nginx.conf
```

重点关注：

```text
Modify
Change
```

---

### 查看日志文件大小

```bash
stat -c %s app.log
```

---

### 查看文件权限

```bash
stat -c "%A %a" script.sh
```

输出：

```text
-rwxr-xr-x 755
```

---

### 查看 inode（排查硬链接问题）

```bash
stat file1
stat file2
```

如果 inode 相同：

```text
Inode: 123456
```

说明两个文件实际上指向同一个 inode，是硬链接。

---

# 总结

`stat` 可以理解为 **“文件身份证查看器”**：

* 看大小：`stat -c %s file`
* 看权限：`stat -c %A file`
* 看所有者：`stat -c %U file`
* 看 inode：`stat -c %i file`
* 看修改时间：`stat -c %y file`
* 看文件系统：`stat -f /`

在日常运维和故障排查中，`stat` 是分析文件属性、时间戳和权限问题非常常用的工具。
