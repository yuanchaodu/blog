---
title: Linux dnf 命令
section: IT
category: Linux
---

# Linux dnf 命令

<img src="images/Linux.svg" width="300">

`dnf`（Dandified YUM）是目前许多 Linux 发行版（如 Fedora、Rocky Linux、AlmaLinux、RHEL 8及以上版本）的软件包管理工具，用来安装、更新、删除和查询软件。

可以把它理解为 Linux 系统中的“应用商店管理器”。

---

# 一、基本语法

```bash
dnf [选项] [命令] [软件包]
```

例如：

```bash
dnf install nginx
```

安装 nginx 软件。

---

# 二、常用命令

## 1. 查看仓库中的软件

搜索软件：

```bash
dnf search nginx
```

查看软件详细信息：

```bash
dnf info nginx
```

示例输出：

```bash
Name         : nginx
Version      : 1.26.0
Release      : 1
Architecture : x86_64
Summary      : High performance web server
```

---

## 2. 安装软件

安装单个软件：

```bash
dnf install nginx
```

安装多个软件：

```bash
dnf install nginx git wget
```

自动确认：

```bash
dnf install -y nginx
```

---

## 3. 更新软件

更新指定软件：

```bash
dnf update nginx
```

或：

```bash
dnf upgrade nginx
```

更新全部软件：

```bash
dnf update
```

或：

```bash
dnf upgrade
```

---

## 4. 删除软件

删除软件：

```bash
dnf remove nginx
```

删除多个软件：

```bash
dnf remove nginx httpd
```

---

## 5. 查询已安装软件

查看是否安装：

```bash
dnf list installed nginx
```

查看所有已安装软件：

```bash
dnf list installed
```

过滤查询：

```bash
dnf list installed | grep nginx
```

---

## 6. 查看可更新的软件

```bash
dnf check-update
```

输出示例：

```bash
nginx.x86_64      1.26.1
git.x86_64        2.45.0
```

表示有新版本可升级。

---

# 三、软件组管理

查看软件组：

```bash
dnf grouplist
```

安装软件组：

```bash
dnf groupinstall "Development Tools"
```

安装开发工具集：

```bash
dnf groupinstall "Development Tools"
```

通常会安装：

* gcc
* g++
* make
* gdb
* autoconf

删除软件组：

```bash
dnf groupremove "Development Tools"
```

---

# 四、仓库管理

查看仓库：

```bash
dnf repolist
```

查看所有仓库：

```bash
dnf repolist all
```

输出类似：

```text
repo id             repo name
baseos              BaseOS
appstream           AppStream
epel                EPEL
```

---

# 五、缓存管理

清理缓存：

```bash
dnf clean all
```

重新生成缓存：

```bash
dnf makecache
```

查看缓存大小：

```bash
du -sh /var/cache/dnf
```

---

# 六、查看依赖关系

查看软件依赖：

```bash
dnf deplist nginx
```

查询某个文件属于哪个软件包：

```bash
dnf provides /usr/bin/python3
```

或者：

```bash
dnf whatprovides /usr/bin/python3
```

这是排查系统问题时非常常用的命令。

---

# 七、历史记录管理

查看安装历史：

```bash
dnf history
```

查看某次操作：

```bash
dnf history info 10
```

回滚操作：

```bash
dnf history undo 10
```

回滚到某个历史点：

```bash
dnf history rollback 10
```

---

# 八、离线下载安装包

仅下载不安装：

```bash
dnf download nginx
```

需要安装插件：

```bash
dnf install 'dnf-command(download)'
```

下载 RPM 及依赖：

```bash
dnf download --resolve nginx
```

---

# 九、系统管理员常用命令

更新系统：

```bash
dnf update -y
```

安装常用工具：

```bash
dnf install -y vim wget curl net-tools
```

安装开发环境：

```bash
dnf groupinstall -y "Development Tools"
```

安装 EPEL 仓库：

对于 RHEL/Rocky/AlmaLinux：

```bash
dnf install epel-release
```

重新生成仓库缓存：

```bash
dnf clean all
dnf makecache
```

查看最近安装的软件：

```bash
dnf history list
```

---

# 十、与 yum 的关系

在 RHEL 8、Rocky Linux 8、AlmaLinux 8 以后：

```text
yum → dnf
```

大部分命令完全兼容：

```bash
yum install nginx
```

等价于：

```bash
dnf install nginx
```

实际上很多系统中的 `yum` 只是指向 `dnf` 的兼容链接。

查看：

```bash
ls -l /usr/bin/yum
```

通常会看到：

```text
/usr/bin/yum -> dnf
```

---

## 常用命令速查表

| 功能      | 命令                    |
| ------- | --------------------- |
| 搜索软件    | `dnf search 软件名`      |
| 查看信息    | `dnf info 软件名`        |
| 安装软件    | `dnf install 软件名`     |
| 删除软件    | `dnf remove 软件名`      |
| 更新软件    | `dnf update 软件名`      |
| 更新系统    | `dnf update`          |
| 查看已安装软件 | `dnf list installed`  |
| 查看可升级软件 | `dnf check-update`    |
| 查看仓库    | `dnf repolist`        |
| 清理缓存    | `dnf clean all`       |
| 重建缓存    | `dnf makecache`       |
| 查询文件归属  | `dnf provides 文件路径`   |
| 查看历史    | `dnf history`         |
| 回滚操作    | `dnf history undo ID` |

对于企业环境（如 Rocky Linux、AlmaLinux、RHEL 服务器），日常使用频率最高的通常是：

```bash
dnf install
dnf remove
dnf update
dnf search
dnf info
dnf repolist
dnf clean all
dnf makecache
dnf history
dnf provides
```

掌握这几个命令，基本可以完成绝大多数服务器软件管理工作。