---
title: Linux hostnamectl 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnpJB
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/150'
---

# Linux hostnamectl 命令

<img src="images/Linux.svg" width="300">

`hostnamectl` 是 Linux 中用于**查看和管理主机名及主机信息**的命令，主要用于使用 `systemd` 的发行版，如 Ubuntu、Debian、RHEL、Rocky Linux、CentOS 等。

## 1. 查看主机信息

```bash
hostnamectl
```

典型输出：

```text
 Static hostname: server01
       Icon name: computer-server
         Chassis: server
      Machine ID: ...
         Boot ID: ...
Operating System: Rocky Linux 9
          Kernel: Linux 5.14.0
    Architecture: x86-64
```

其中最常关注的是：

* `Static hostname`：永久主机名
* `Operating System`：操作系统
* `Kernel`：Linux 内核版本
* `Architecture`：CPU 架构

## 2. 修改主机名

```bash
sudo hostnamectl set-hostname 新主机名
```

例如：

```bash
sudo hostnamectl set-hostname app-server01
```

查看是否修改成功：

```bash
hostnamectl
```

或：

```bash
hostname
```

修改通常会立即生效，但已有的终端提示符可能不会马上变化，重新登录后即可看到新主机名。

## 3. 查看指定信息

只查看当前主机名：

```bash
hostnamectl hostname
```

部分较旧的系统可以使用：

```bash
hostnamectl --static
```

查看系统状态：

```bash
hostnamectl status
```

## 4. 三种主机名的区别

`hostnamectl` 可以管理三种主机名：

| 类型        | 含义          | 示例                      |
| --------- | ----------- | ----------------------- |
| Static    | 永久主机名，最常用   | `server01`              |
| Transient | 临时主机名       | DHCP 动态分配的名称            |
| Pretty    | 便于人阅读的描述性名称 | `Production Web Server` |

设置永久主机名：

```bash
sudo hostnamectl set-hostname server01
```

设置友好名称：

```bash
sudo hostnamectl set-hostname "Production Web Server" --pretty
```

## 5. 常用命令速查

```bash
# 查看主机和系统信息
hostnamectl

# 修改永久主机名
sudo hostnamectl set-hostname server01

# 查看当前主机名
hostname

# 查看内核信息
uname -r

# 查看完整内核和架构信息
uname -a
```

需要注意的是，修改主机名后，最好检查 `/etc/hosts` 中是否仍然保留旧主机名。例如：

```text
127.0.0.1   localhost
127.0.1.1   server01
```

在服务器环境中，建议同时检查 DNS、监控系统、备份系统和应用配置是否使用了旧主机名，避免改名后出现连接或识别问题。
