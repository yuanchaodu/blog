---
title: Linux 用户组管理
section: IT
category: Linux
---

# Linux 用户组管理

<img src="images/Linux.svg" width="300">

Linux 用户组管理主要用于**控制用户权限**。可以把用户组理解为“部门”：同一部门的人，可以统一分配访问权限。

## 1. 查看用户和组

```bash
id 用户名
```

查看某个用户属于哪些组。

```bash
groups 用户名
```

只查看用户所在组。

```bash
cat /etc/group
```

查看系统所有组。

## 2. 创建用户组

```bash
sudo groupadd 组名
```

例如：

```bash
sudo groupadd dev
```

## 3. 删除用户组

```bash
sudo groupdel 组名
```

例如：

```bash
sudo groupdel dev
```

注意：如果某个用户的**主组**是该组，通常不能直接删除。

## 4. 修改用户组名

```bash
sudo groupmod -n 新组名 旧组名
```

例如：

```bash
sudo groupmod -n developers dev
```

## 5. 把用户加入附加组

```bash
sudo usermod -aG 组名 用户名
```

例如：

```bash
sudo usermod -aG dev zhangsan
```

注意：`-aG` 很重要。
少了 `-a`，可能会覆盖用户原来的附加组。

## 6. 从用户组中移除用户

Debian/Ubuntu 常用：

```bash
sudo deluser 用户名 组名
```

通用方式：

```bash
sudo gpasswd -d 用户名 组名
```

例如：

```bash
sudo gpasswd -d zhangsan dev
```

## 7. 修改用户主组

```bash
sudo usermod -g 组名 用户名
```

例如：

```bash
sudo usermod -g dev zhangsan
```

## 8. 查看文件所属用户和组

```bash
ls -l 文件名
```

示例：

```bash
-rw-r--r-- 1 zhangsan dev 1234 file.txt
```

其中：

```text
zhangsan 是所属用户
dev 是所属组
```

## 9. 修改文件所属组

```bash
sudo chgrp 组名 文件名
```

例如：

```bash
sudo chgrp dev file.txt
```

递归修改目录：

```bash
sudo chgrp -R dev /data/project
```

## 10. 常见权限配合

让某个组可以读写目录：

```bash
sudo chgrp -R dev /data/project
sudo chmod -R 770 /data/project
```

含义：

```text
所属用户：读、写、执行
所属组：读、写、执行
其他用户：无权限
```

常用排查命令：

```bash
id 用户名
ls -ld 目录名
namei -l 路径
```

`namei -l` 可以逐级查看目录权限，排查“明明在组里却访问不了”的问题。

