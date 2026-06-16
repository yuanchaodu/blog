---
title: Linux sudo 命令
section: IT
category: Linux
---

# Linux sudo 命令

<img src="images/Linux.svg" width="300">

`sudo` 是 Linux 中常用的提权命令，意思是 **以管理员或其他用户身份执行命令**。

## 1. 基本作用

普通用户权限有限，很多系统操作不能直接执行，比如安装软件、修改系统配置、重启服务等。

使用 `sudo` 后，可以临时获得管理员权限：

```bash
sudo 命令
```

例如：

```bash
sudo apt update
```

表示用管理员权限更新软件源。

## 2. 常见用法

安装软件：

```bash
sudo apt install nginx
```

编辑系统文件：

```bash
sudo nano /etc/hosts
```

重启服务：

```bash
sudo systemctl restart nginx
```

查看需要管理员权限的文件：

```bash
sudo cat /var/log/syslog
```

切换到 root 用户：

```bash
sudo -i
```

或：

```bash
sudo su -
```

## 3. 使用特点

第一次执行 `sudo` 时，系统通常会要求输入当前用户的密码。

注意：输入密码时终端不会显示星号，这是正常现象。

认证成功后，短时间内再次使用 `sudo` 通常不需要重新输入密码。

## 4. 权限配置

能不能使用 `sudo`，取决于用户是否在 sudo 权限范围内。

常见配置文件是：

```bash
/etc/sudoers
```

不要直接用普通编辑器修改它，建议使用：

```bash
sudo visudo
```

这样可以避免语法错误导致 sudo 失效。

## 5. 常见问题

如果提示：

```bash
user is not in the sudoers file
```

说明当前用户没有 sudo 权限，需要管理员把该用户加入 sudo 组。

Ubuntu/Debian 常见命令：

```bash
sudo usermod -aG sudo 用户名
```

CentOS/RHEL 常见命令：

```bash
sudo usermod -aG wheel 用户名
```

## 6. 简单理解

可以把 `sudo` 理解为“临时拿到系统管理员钥匙”。

平时不要长期使用 root 用户，日常操作用普通用户，需要管理权限时再加 `sudo`，这样更安全。