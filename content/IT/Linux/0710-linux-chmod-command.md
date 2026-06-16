---
title: Linux chmod 命令
section: IT
category: Linux
---

# Linux chmod 命令

<img src="images/Linux.svg" width="300">

`chmod` 是 Linux 中用来**修改文件或目录权限**的命令。

## 1. 基本格式

```bash
chmod 权限 文件名
```

例如：

```bash
chmod 755 test.sh
```

表示给 `test.sh` 设置权限。

## 2. Linux 权限含义

Linux 文件权限分三类用户：

| 类别 | 含义    |
| -- | ----- |
| u  | 文件所有者 |
| g  | 所属用户组 |
| o  | 其他用户  |
| a  | 所有人   |

权限有三种：

| 权限 | 含义 | 数字 |
| -- | -- | -- |
| r  | 读取 | 4  |
| w  | 写入 | 2  |
| x  | 执行 | 1  |

## 3. 数字权限写法

常见权限：

```bash
chmod 755 file
```

含义：

```text
所有者：读、写、执行 = 7
用户组：读、执行 = 5
其他人：读、执行 = 5
```

常见示例：

```bash
chmod 644 file.txt
```

普通文件常用，所有者可读写，其他人只读。

```bash
chmod 755 script.sh
```

脚本或程序常用，所有者可读写执行，其他人可读可执行。

```bash
chmod 700 private.sh
```

只有所有者可以读、写、执行。

## 4. 字母权限写法

给所有者增加执行权限：

```bash
chmod u+x file.sh
```

去掉其他人的写权限：

```bash
chmod o-w file.txt
```

给所有人增加读取权限：

```bash
chmod a+r file.txt
```

设置用户组可读写：

```bash
chmod g=rw file.txt
```

## 5. 修改目录权限

```bash
chmod 755 mydir
```

目录中的 `x` 表示可以进入目录。

递归修改目录及其下面所有文件：

```bash
chmod -R 755 mydir
```

注意：`-R` 要小心使用，尤其不要随意对系统目录执行。

## 6. 常用建议

普通文件常用：

```bash
chmod 644 filename
```

脚本文件常用：

```bash
chmod 755 script.sh
```

私密文件常用：

```bash
chmod 600 secret.txt
```

私密目录常用：

```bash
chmod 700 private_dir
```

一句话理解：`chmod` 就是给文件和目录“改门锁”，决定谁能看、谁能改、谁能执行。