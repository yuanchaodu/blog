---
title: Linux 用户管理
section: IT
category: Linux
---

# Linux 用户管理

<img src="images/Linux.svg" width="300">

Linux 用户管理主要包括 **用户、用户组、权限、密码和切换身份** 这几部分。

## 1. 用户相关命令

创建用户：

```bash
useradd username
```

创建用户并生成家目录：

```bash
useradd -m username
```

设置密码：

```bash
passwd username
```

删除用户：

```bash
userdel username
```

删除用户并删除家目录：

```bash
userdel -r username
```

查看用户信息：

```bash
id username
```

查看当前登录用户：

```bash
who
```

查看当前用户名：

```bash
whoami
```

## 2. 用户组相关命令

创建用户组：

```bash
groupadd groupname
```

删除用户组：

```bash
groupdel groupname
```

把用户加入用户组：

```bash
usermod -aG groupname username
```

查看用户所属组：

```bash
groups username
```

## 3. 常用配置文件

用户信息：

```bash
/etc/passwd
```

密码信息：

```bash
/etc/shadow
```

用户组信息：

```bash
/etc/group
```

## 4. 切换用户

切换到某个用户：

```bash
su - username
```

使用管理员权限执行命令：

```bash
sudo command
```

切换到 root：

```bash
sudo -i
```

## 5. 权限管理

查看文件权限：

```bash
ls -l
```

修改文件所有者：

```bash
chown username filename
```

修改文件所属组：

```bash
chgrp groupname filename
```

修改权限：

```bash
chmod 755 filename
```

常见权限含义：

```text
r：读
w：写
x：执行
```

## 6. 示例

创建一个用户并加入指定组：

```bash
groupadd appgroup
useradd -m -s /bin/bash appuser
passwd appuser
usermod -aG appgroup appuser
```

Linux 用户管理的核心思路是：
**用用户区分身份，用用户组统一授权，用权限控制资源访问。**