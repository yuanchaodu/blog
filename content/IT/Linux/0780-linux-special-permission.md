---
title: Linux 特殊权限位 SUID SGID Sticky
section: IT
category: Linux
---

# Linux 特殊权限位 SUID SGID Sticky

<img src="images/Linux.svg" width="300">

Linux 中的 **SUID、SGID 和 Sticky Bit** 是三种特殊权限位，用于在传统 `rwx` 权限之外实现更灵活的访问控制。

## 一、查看特殊权限

普通权限：

```bash
-rwxr-xr-x
```

带特殊权限时：

```bash
-rwsr-xr-x   # SUID
-rwxr-sr-x   # SGID
drwxrwxrwt   # Sticky Bit
```

使用数字表示：

| 权限         | 数值 |
| ---------- | -- |
| SUID       | 4  |
| SGID       | 2  |
| Sticky Bit | 1  |

例如：

```bash
chmod 4755 file
chmod 2755 dir
chmod 1777 dir
```

---

# 二、SUID（Set User ID）

## 基本作用

当普通用户执行一个带 SUID 的程序时：

**程序运行时使用文件所有者的身份，而不是执行者的身份。**

### 示例

查看 passwd 命令：

```bash
ls -l /usr/bin/passwd
```

输出类似：

```bash
-rwsr-xr-x 1 root root 68248 Jan 10 10:00 /usr/bin/passwd
```

注意：

```bash
rws
```

其中的：

```bash
s
```

表示 SUID。

---

### 为什么需要 SUID？

普通用户修改密码：

```bash
passwd
```

实际上需要修改：

```bash
/etc/shadow
```

而该文件权限：

```bash
-r-------- root root
```

普通用户无权修改。

但由于：

```bash
passwd
```

带有 SUID，

执行时临时获得：

```bash
root 身份
```

因此能够修改密码。

---

### 设置 SUID

```bash
chmod u+s file
```

或：

```bash
chmod 4755 file
```

查看：

```bash
ls -l file
```

结果：

```bash
-rwsr-xr-x
```

---

# 三、SGID（Set Group ID）

SGID 在文件和目录上的作用不同。

---

## 1. SGID 用于文件

执行程序时：

**进程获得文件所属组权限。**

设置：

```bash
chmod g+s file
```

或：

```bash
chmod 2755 file
```

查看：

```bash
-rwxr-sr-x
```

---

## 2. SGID 用于目录（最常见）

### 功能

目录中新建文件自动继承目录所属组。

---

### 示例

创建共享目录：

```bash
mkdir /share
chgrp project /share
chmod 2775 /share
```

查看：

```bash
drwxrwsr-x
```

此时：

```bash
user1 创建文件
user2 创建文件
```

文件都会属于：

```bash
project
```

组。

---

### 不设置 SGID

默认：

```bash
新文件属于用户自己的主组
```

例如：

```bash
user1 -> dev
user2 -> test
```

创建文件后组可能不一致。

---

### 设置 SGID 后

统一变成：

```bash
project
```

适合：

* 项目共享目录
* 开发团队目录
* 部门共享空间

---

# 四、Sticky Bit（粘滞位）

主要用于目录。

---

## 功能

目录所有人都有写权限时：

**只能删除自己拥有的文件。**

---

## 示例

系统临时目录：

```bash
ls -ld /tmp
```

输出：

```bash
drwxrwxrwt
```

最后的：

```bash
t
```

就是 Sticky Bit。

---

### 场景

假设：

```bash
/tmp
```

权限：

```bash
777
```

用户：

```bash
zhangsan
lisi
```

都能写入。

如果没有 Sticky Bit：

```bash
lisi 可以删除 zhangsan 的文件
```

存在安全风险。

---

### 有 Sticky Bit

```bash
drwxrwxrwt
```

则：

```bash
zhangsan 只能删自己的文件
lisi 只能删自己的文件
root 可删除所有文件
```

---

### 设置 Sticky Bit

```bash
chmod +t dir
```

或：

```bash
chmod 1777 dir
```

查看：

```bash
drwxrwxrwt
```

---

# 五、大小写区别

查看权限时可能看到：

```bash
s
S
t
T
```

区别：

| 显示 | 含义                   |
| -- | -------------------- |
| s  | 特殊权限+执行权限都有          |
| S  | 只有特殊权限，没有执行权限        |
| t  | Sticky Bit+执行权限都有    |
| T  | 只有 Sticky Bit，没有执行权限 |

例如：

```bash
-rwSr--r--
```

表示：

```bash
SUID 已设置
但执行权限 x 未设置
```

---

# 六、数字表示法

权限位：

```bash
SUID   = 4
SGID   = 2
Sticky = 1
```

组合计算：

| 权限        | 数字   |
| --------- | ---- |
| SUID      | 4xxx |
| SGID      | 2xxx |
| Sticky    | 1xxx |
| SUID+SGID | 6xxx |
| 三个都开      | 7xxx |

示例：

```bash
4755  # SUID
2755  # SGID
1777  # Sticky
6755  # SUID+SGID
7755  # 全部
```

---

# 七、实际工作中的典型应用

### 1. passwd 命令

```bash
-rwsr-xr-x root root /usr/bin/passwd
```

使用 SUID。

---

### 2. 项目共享目录

```bash
mkdir /data/project
chgrp devteam /data/project
chmod 2775 /data/project
```

使用 SGID。

---

### 3. 临时文件目录

```bash
chmod 1777 /tmp
```

使用 Sticky Bit。

---

## 总结

| 特殊权限       | 作用对象  | 功能            |
| ---------- | ----- | ------------- |
| SUID       | 可执行文件 | 以文件所有者身份运行    |
| SGID       | 文件    | 以文件所属组身份运行    |
| SGID       | 目录    | 新文件继承目录所属组    |
| Sticky Bit | 目录    | 只能删除自己的文件     |
| ACL        | 文件/目录 | 对特定用户或组设置精细权限 |

在企业 Linux 环境中，**SGID + ACL** 经常一起使用来建设共享目录；而 **SUID** 由于存在提权风险，应严格控制，仅保留系统必要程序使用。