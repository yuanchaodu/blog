---
title: Linux 文件属性管理
section: IT
category: Linux
---

# Linux 文件属性管理

<img src="images/Linux.svg" width="300">

下面是 **Linux 文件属性管理** 的核心内容，适合入门和日常运维使用。

## 1. 查看文件属性

常用命令：

```bash
ls -l
```

示例：

```bash
-rw-r--r-- 1 root root 1024 Jun 16  test.txt
```

含义：

```text
-rw-r--r--   文件权限
1            硬链接数量
root         文件所有者
root         所属用户组
1024         文件大小
Jun 16       修改时间
test.txt     文件名
```

查看更详细属性：

```bash
stat test.txt
```

## 2. 文件类型

`ls -l` 第一位表示文件类型：

| 标识  | 含义   |
| --- | ---- |
| `-` | 普通文件 |
| `d` | 目录   |
| `l` | 软链接  |
| `c` | 字符设备 |
| `b` | 块设备  |
| `s` | 套接字  |
| `p` | 管道文件 |

## 3. 文件权限

Linux 权限分为三类：

| 对象     | 含义   |
| ------ | ---- |
| user   | 所有者  |
| group  | 所属组  |
| others | 其他用户 |

权限有三种：

| 权限 | 含义 | 数字 |
| -- | -- | -- |
| r  | 读  | 4  |
| w  | 写  | 2  |
| x  | 执行 | 1  |

例如：

```text
-rw-r--r--
```

表示：

```text
所有者：可读、可写
所属组：可读
其他人：可读
```

## 4. 修改权限：chmod

使用字符方式：

```bash
chmod u+x script.sh
chmod g-w file.txt
chmod o+r file.txt
```

使用数字方式：

```bash
chmod 755 script.sh
chmod 644 file.txt
```

常见权限：

| 权限  | 说明               |
| --- | ---------------- |
| 755 | 所有者可读写执行，其他人可读执行 |
| 644 | 所有者可读写，其他人只读     |
| 700 | 只有所有者可读写执行       |
| 600 | 只有所有者可读写         |

## 5. 修改所有者：chown

修改文件所有者：

```bash
chown user file.txt
```

修改所有者和用户组：

```bash
chown user:group file.txt
```

递归修改目录：

```bash
chown -R user:group /data
```

## 6. 修改所属组：chgrp

```bash
chgrp group file.txt
```

递归修改：

```bash
chgrp -R group /data
```

## 7. 特殊权限

Linux 还有三类特殊权限：

| 权限         | 作用                     |
| ---------- | ---------------------- |
| SUID       | 普通用户执行文件时，临时拥有文件所有者权限  |
| SGID       | 文件按所属组权限执行；目录中新文件继承目录组 |
| Sticky Bit | 常用于共享目录，用户只能删除自己的文件    |

示例：

```bash
chmod u+s file
chmod g+s dir
chmod +t /shared
```

常见例子：

```bash
/tmp
```

通常带有 Sticky Bit：

```bash
drwxrwxrwt
```

## 8. 文件隐藏属性：chattr

查看隐藏属性：

```bash
lsattr file.txt
```

设置文件不可修改：

```bash
chattr +i file.txt
```

取消不可修改：

```bash
chattr -i file.txt
```

常用属性：

| 属性 | 含义             |
| -- | -------------- |
| i  | 不可修改、删除、重命名    |
| a  | 只能追加内容，不能覆盖或删除 |

## 9. 默认权限：umask

查看默认权限掩码：

```bash
umask
```

常见值：

```bash
0022
```

通常表示：

```text
新建文件默认权限：644
新建目录默认权限：755
```

临时修改：

```bash
umask 027
```

## 10. 常用排查命令

```bash
ls -l file
stat file
id user
groups user
getfacl file
lsattr file
```

## 小结

Linux 文件属性管理主要包括：

```text
查看属性：ls -l、stat
修改权限：chmod
修改所有者：chown
修改用户组：chgrp
管理特殊权限：SUID、SGID、Sticky Bit
管理隐藏属性：chattr、lsattr
控制默认权限：umask
```

日常运维中，最常用的是：

```bash
ls -l
chmod
chown
chgrp
umask
```