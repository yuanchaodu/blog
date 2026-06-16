---
title: Linux chgrp 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKO0
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/73'
---

# Linux chgrp 命令

<img src="images/Linux.svg" width="300">

`chgrp`（change group）是 Linux/Unix 系统中用于**修改文件或目录所属用户组**的命令。

## 基本语法

```bash
chgrp [选项] 用户组 文件或目录
```

例如：

```bash
chgrp developers test.txt
```

表示将 `test.txt` 的所属组修改为 `developers`。

---

## 常用示例

### 1. 修改单个文件的所属组

```bash
chgrp staff file.txt
```

执行前：

```bash
ls -l file.txt
```

输出：

```text
-rw-r--r-- 1 nick users 1024 Jun 16 10:00 file.txt
```

执行后：

```text
-rw-r--r-- 1 nick staff 1024 Jun 16 10:00 file.txt
```

所属组从 `users` 变成了 `staff`。

---

### 2. 递归修改目录及其所有内容

```bash
chgrp -R staff /data/project
```

其中：

```text
-R = Recursive（递归）
```

会修改：

* project目录
* project下所有子目录
* project下所有文件

---

### 3. 根据参考文件修改组

```bash
chgrp --reference=file1.txt file2.txt
```

表示：

```text
让 file2.txt 的所属组与 file1.txt 保持一致
```

---

## 常用参数

| 参数               | 说明         |
| ---------------- | ---------- |
| `-R`             | 递归修改目录     |
| `-v`             | 显示修改过程     |
| `-c`             | 仅显示发生变化的文件 |
| `-f`             | 忽略错误信息     |
| `--reference=文件` | 参考指定文件的组属性 |

例如：

```bash
chgrp -Rv developers /app
```

输出类似：

```text
changed group of '/app/test.txt' from 'users' to 'developers'
changed group of '/app/log' from 'users' to 'developers'
```

---

## 权限要求

一般情况下：

* 文件所有者可以将文件组改为自己所属的组
* root 用户可以修改为任意组

查看当前用户所属组：

```bash
groups
```

或：

```bash
id
```

例如：

```text
uid=1001(nick)
gid=100(users)
groups=100(users),101(staff),102(developers)
```

则可以把文件组改成：

* users
* staff
* developers

不能改成不属于自己的组（除非使用 root）。

---

## chgrp 与 chown 的区别

| 命令      | 功能        |
| ------- | --------- |
| `chgrp` | 修改所属组     |
| `chown` | 修改所有者和所属组 |

例如：

```bash
chgrp developers file.txt
```

仅修改组。

```bash
chown tom file.txt
```

修改所有者。

```bash
chown tom:developers file.txt
```

同时修改所有者和组。

---

## 实际运维中的常见用法

### Web目录授权

```bash
chgrp -R www-data /var/www/html
```

让 Web 服务组拥有目录权限。

### 项目共享目录

```bash
chgrp -R projectteam /data/project
chmod -R 775 /data/project
```

让同组成员都可以读写项目文件。

### 查看文件所属组

```bash
ls -l
```

例如：

```text
-rw-r--r-- 1 root developers 2048 Jun 16 10:00 app.log
```

其中：

```text
root        → 文件所有者
developers  → 文件所属组
```

---

## 注意事项

如果出现：

```text
chgrp: changing group of 'file.txt': Operation not permitted
```

通常有以下原因：

1. 当前用户不是文件所有者。
2. 目标组不属于当前用户。
3. 文件位于只读文件系统。
4. 需要使用 `sudo`。

例如：

```bash
sudo chgrp developers file.txt
```

这是 Linux 系统管理、共享目录权限控制和应用部署中经常使用的基础命令。
