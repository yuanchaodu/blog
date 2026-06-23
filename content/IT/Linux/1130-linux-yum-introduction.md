---
title: Linux yum 介绍
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnUN5
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/113'
---

# Linux yum 介绍

<img src="images/Linux.svg" width="300">

## 什么是 YUM

YUM（**Yellowdog Updater, Modified**）是 Linux 系统中常用的软件包管理工具，主要用于基于 RPM 包管理机制的发行版，例如：

* CentOS
* Red Hat Enterprise Linux（RHEL）
* Oracle Linux
* Rocky Linux
* AlmaLinux

YUM 的作用类似于手机应用商店，可以自动完成：

* 软件安装
* 软件升级
* 软件卸载
* 软件依赖关系处理
* 软件仓库管理

例如，安装一个软件时，如果它依赖其他组件，YUM 会自动下载并安装这些依赖包。

---

## YUM 工作原理

YUM 的工作流程如下：

```text
用户执行 yum 命令
        ↓
读取仓库配置文件
        ↓
访问软件仓库（Repository）
        ↓
获取软件包及依赖信息
        ↓
自动下载安装
        ↓
更新本地数据库
```

简单来说：

```text
YUM = RPM + 软件仓库 + 自动依赖管理
```

其中：

* RPM负责软件包格式
* Repository负责存放软件
* YUM负责管理整个过程

---

## YUM 与 RPM 的区别

| 项目     | RPM | YUM |
| ------ | --- | --- |
| 安装软件   | √   | √   |
| 卸载软件   | √   | √   |
| 查询软件   | √   | √   |
| 自动处理依赖 | ×   | √   |
| 在线下载软件 | ×   | √   |
| 仓库管理   | ×   | √   |

例如：

RPM安装：

```bash
rpm -ivh nginx.rpm
```

如果缺少依赖：

```text
error: Failed dependencies
```

YUM安装：

```bash
yum install nginx
```

YUM自动解决依赖问题。

---

## YUM 仓库（Repository）

仓库是存放 RPM 软件包的服务器。

常见仓库：

### Base仓库

系统官方基础软件仓库。

### EPEL仓库

EPEL（Extra Packages for Enterprise Linux）由：

Fedora Project

维护，提供大量额外软件。

安装EPEL：

```bash
yum install epel-release
```

---

## YUM 配置文件

主配置文件：

```bash
/etc/yum.conf
```

仓库配置目录：

```bash
/etc/yum.repos.d/
```

例如：

```bash
ls /etc/yum.repos.d/
```

输出：

```text
CentOS-Base.repo
epel.repo
```

查看仓库配置：

```bash
cat /etc/yum.repos.d/CentOS-Base.repo
```

典型配置：

```ini
[base]
name=CentOS Base
baseurl=http://mirror.centos.org
enabled=1
gpgcheck=1
```

参数说明：

| 参数       | 含义     |
| -------- | ------ |
| name     | 仓库名称   |
| baseurl  | 仓库地址   |
| enabled  | 是否启用   |
| gpgcheck | 是否校验签名 |

---

## 常用 YUM 命令

### 1. 查看仓库

```bash
yum repolist
```

显示启用的仓库：

```text
repo id      repo name
base         CentOS Base
epel         Extra Packages
```

---

### 2. 搜索软件

搜索 nginx：

```bash
yum search nginx
```

搜索 mysql：

```bash
yum search mysql
```

---

### 3. 查看软件信息

```bash
yum info nginx
```

显示：

```text
Name        : nginx
Version     : 1.24.0
Release     : 1
Architecture: x86_64
```

---

### 4. 安装软件

```bash
yum install nginx
```

自动确认：

```bash
yum install -y nginx
```

安装多个软件：

```bash
yum install -y vim wget net-tools
```

---

### 5. 升级软件

升级指定软件：

```bash
yum update nginx
```

升级全部软件：

```bash
yum update
```

---

### 6. 卸载软件

```bash
yum remove nginx
```

或者：

```bash
yum erase nginx
```

---

### 7. 查看已安装软件

```bash
yum list installed
```

查看指定软件：

```bash
yum list installed nginx
```

---

### 8. 查看可更新软件

```bash
yum check-update
```

---

### 9. 查看软件依赖

```bash
yum deplist nginx
```

输出：

```text
dependency: libc.so.6
dependency: openssl
dependency: pcre
```

---

### 10. 查看软件来源仓库

```bash
yum list nginx
```

或：

```bash
yum provides */nginx
```

---

## YUM 缓存管理

YUM会把下载的软件包缓存到本地。

缓存目录：

```bash
/var/cache/yum/
```

查看缓存：

```bash
yum clean expire-cache
```

清理缓存：

```bash
yum clean all
```

重新生成缓存：

```bash
yum makecache
```

---

## 软件包组管理

查看软件组：

```bash
yum grouplist
```

安装开发工具组：

```bash
yum groupinstall "Development Tools"
```

常用于编译环境搭建。

例如安装：

* gcc
* g++
* make
* gdb

等工具。

---

## 历史记录管理

查看操作历史：

```bash
yum history
```

示例：

```text
ID | Command
1  | install nginx
2  | update
```

查看某次操作：

```bash
yum history info 1
```

回滚操作：

```bash
yum history undo 1
```

这在生产环境中非常有用。

---

## YUM 与 DNF

从 Fedora 22开始以及后续的 RHEL8/CentOS8 系列中，YUM 已逐渐被 DNF（Dandified YUM）取代。

关系如下：

```text
YUM (CentOS 7)
       ↓
DNF (CentOS 8/Rocky 8/RHEL 8)
```

实际上：

```bash
yum install nginx
```

在很多新版本系统中，执行的底层已经是：

```bash
dnf install nginx
```

常见对应关系：

| YUM         | DNF         |
| ----------- | ----------- |
| yum install | dnf install |
| yum update  | dnf update  |
| yum remove  | dnf remove  |
| yum search  | dnf search  |
| yum info    | dnf info    |

---

## 企业运维中的常见场景

### 安装运维工具

```bash
yum install -y vim wget curl net-tools
```

### 安装 Web 服务

```bash
yum install -y nginx
```

### 安装数据库

```bash
yum install -y mariadb-server
```

### 安装开发环境

```bash
yum groupinstall "Development Tools"
```

### 构建本地 YUM 仓库

对于化工企业、制造企业等内网环境，经常会搭建本地 YUM 镜像仓库，实现：

* 统一软件源管理
* 提高安装速度
* 减少外网访问
* 满足网络安全要求
* 支持批量服务器运维

常见方案包括：

* reposync 同步官方仓库
* createrepo 创建仓库元数据
* 使用 Nginx 或 Apache HTTP Server 发布仓库

---

## 总结

YUM 的核心价值可以概括为：

```text
RPM负责安装
YUM负责管理
仓库负责提供软件
```

掌握以下几个命令基本就能满足日常运维需求：

```bash
yum repolist              # 查看仓库
yum search 软件名         # 搜索软件
yum info 软件名           # 查看信息
yum install 软件名        # 安装
yum update                # 更新
yum remove 软件名         # 卸载
yum history               # 查看历史
yum clean all             # 清理缓存
```

如果您日常维护的是 CentOS 7、RHEL 7 等服务器，YUM 仍然是最重要的软件管理工具之一；如果是 Rocky Linux 8/9、AlmaLinux 8/9、RHEL 8/9，则建议同时掌握 DNF，因为它已经成为新一代标准包管理工具。
