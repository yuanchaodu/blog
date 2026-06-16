---
title: Linux chattr 与 lsattr 命令
section: IT
category: Linux
---

# Linux chattr 与 lsattr 命令

<img src="images/Linux.svg" width="300">

在 Linux 中，`chattr` 和 `lsattr` 是一组用于管理文件**扩展属性（Extended Attributes）**的工具，主要应用于 `ext2/ext3/ext4` 文件系统，也部分支持其他文件系统。

它们的作用类似于 Windows 中的“只读”“隐藏”等属性，但功能更强大，可以直接影响文件的修改、删除和访问行为。

---

# 一、lsattr：查看文件属性

`lsattr` 用于查看文件或目录的特殊属性。

## 基本语法

```bash
lsattr 文件名
```

例如：

```bash
lsattr test.txt
```

输出：

```bash
----i---------e---- test.txt
```

其中：

```text
i = immutable（不可变）
e = extents（使用 ext4 extent 存储）
```

---

## 查看目录下所有文件

```bash
lsattr /data/*
```

或者递归查看：

```bash
lsattr -R /data
```

---

# 二、chattr：修改文件属性

`chattr`（change attribute）用于设置或取消文件属性。

## 基本语法

```bash
chattr [+|-|=][属性] 文件名
```

说明：

| 符号 | 含义   |
| -- | ---- |
| +  | 增加属性 |
| -  | 移除属性 |
| =  | 指定属性 |

例如：

```bash
chattr +i test.txt
```

给文件增加不可变属性。

---

# 三、最常用的属性

## 1. i（Immutable，不可变）

最常用、最重要的属性。

设置：

```bash
chattr +i test.txt
```

查看：

```bash
lsattr test.txt
```

输出：

```bash
----i---------e---- test.txt
```

---

### 效果

文件不能：

* 修改
* 删除
* 重命名
* 建立硬链接

即使 root 用户也不能直接删除：

```bash
rm test.txt
```

报错：

```text
Operation not permitted
```

---

### 解除

```bash
chattr -i test.txt
```

然后才能删除：

```bash
rm test.txt
```

---

### 典型应用

保护重要配置文件：

```bash
chattr +i /etc/passwd
chattr +i /etc/shadow
```

防止误删：

```bash
chattr +i backup.sql
```

---

## 2. a（Append Only，仅允许追加）

设置：

```bash
chattr +a logfile.log
```

---

### 效果

允许：

```bash
echo "new log" >> logfile.log
```

不允许：

```bash
echo "overwrite" > logfile.log
```

也不允许：

```bash
rm logfile.log
```

---

### 典型应用

日志文件保护：

```bash
/var/log/messages
```

例如：

```bash
chattr +a /var/log/myapp.log
```

日志只能追加，不能被篡改。

---

## 3. A（不更新访问时间）

默认情况下：

```bash
cat file.txt
```

会更新：

```bash
atime
```

设置：

```bash
chattr +A file.txt
```

后：

```bash
cat file.txt
```

不会更新访问时间。

---

### 应用场景

高并发读取目录：

```bash
/var/www/html
```

减少磁盘写入。

---

## 4. d（备份时忽略）

设置：

```bash
chattr +d cache.tmp
```

表示：

```text
dump备份程序忽略该文件
```

---

## 5. S（同步写入）

设置：

```bash
chattr +S important.db
```

每次写入立即同步到磁盘。

类似：

```bash
fsync()
```

---

### 优点

提高数据安全性。

### 缺点

性能下降明显。

---

## 6. s（安全删除）

设置：

```bash
chattr +s secret.txt
```

理论上删除时覆盖原数据。

```bash
rm secret.txt
```

再删除。

不过：

> 现代 SSD、日志文件系统和 ext4 中，该属性实际效果有限，不应作为可靠的数据擦除方案。

---

## 7. u（可恢复删除）

设置：

```bash
chattr +u file.txt
```

理论上保留删除信息用于恢复。

现代文件系统支持有限。

实际生产环境很少使用。

---

# 四、目录属性的特殊作用

## 目录加 i

```bash
chattr +i /data
```

效果：

* 不能创建文件
* 不能删除文件
* 不能修改目录结构

例如：

```bash
touch /data/test
```

报错：

```text
Operation not permitted
```

---

## 目录加 a

```bash
chattr +a /data
```

效果：

* 允许新增文件
* 不允许删除文件

适用于审计目录。

---

# 五、实际运维案例

## 案例1：保护关键配置

```bash
chattr +i /etc/resolv.conf
```

防止 DHCP 自动修改 DNS。

查看：

```bash
lsattr /etc/resolv.conf
```

输出：

```bash
----i---------e----
```

恢复：

```bash
chattr -i /etc/resolv.conf
```

---

## 案例2：保护应用日志

```bash
touch app.log

chattr +a app.log
```

日志只能追加：

```bash
echo test >> app.log
```

但不能覆盖：

```bash
echo test > app.log
```

---

## 案例3：防止误删备份文件

```bash
chattr +i backup.tar.gz
```

任何用户（包括 root）都无法直接删除。

---

# 六、常见问题

## 1. root 为什么删不掉文件？

查看：

```bash
lsattr file.txt
```

如果看到：

```bash
----i---------
```

说明设置了不可变属性。

解除：

```bash
chattr -i file.txt
```

---

## 2. 为什么 chattr 提示 Operation not supported？

例如：

```bash
chattr +i file.txt
```

报错：

```text
Operation not supported
```

通常原因：

* 文件系统不支持
* NFS 挂载目录
* FAT32
* exFAT
* NTFS（部分驱动）

查看文件系统：

```bash
df -T
```

---

## 3. Docker 容器里无法使用 chattr？

很多容器文件系统（如 OverlayFS）不支持全部属性：

```bash
overlay
aufs
```

因此可能报错：

```text
Operation not permitted
```

---

# 七、常用命令速查

| 操作         | 命令                 |
| ---------- | ------------------ |
| 查看属性       | `lsattr file`      |
| 递归查看       | `lsattr -R dir`    |
| 设置不可变      | `chattr +i file`   |
| 取消不可变      | `chattr -i file`   |
| 设置仅追加      | `chattr +a file`   |
| 取消仅追加      | `chattr -a file`   |
| 查看 ext4 属性 | `lsattr file`      |
| 递归设置       | `chattr -R +i dir` |

---

# 八、生产环境建议

对于企业服务器和关键系统，最常见、最有价值的属性实际上只有两个：

| 属性      | 用途          | 推荐程度  |
| ------- | ----------- | ----- |
| `i`     | 防止修改和删除     | ★★★★★ |
| `a`     | 日志防篡改       | ★★★★★ |
| `A`     | 减少 atime 更新 | ★★★☆☆ |
| `S`     | 提高数据安全      | ★★☆☆☆ |
| `s`、`u` | 现代系统意义不大    | ★☆☆☆☆ |

因此在日常运维中，90% 以上的场景都是：

```bash
chattr +i 重要配置文件
```

和

```bash
chattr +a 日志文件
```

这两种用法。它们是 Linux 文件保护中最简单、最有效的手段之一。