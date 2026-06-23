---
title: Linux yum 命令
section: IT
category: Linux
---

# Linux yum 命令

<img src="images/Linux.svg" width="300">

`yum`（Yellowdog Updater, Modified）是 Linux 中常用的软件包管理工具，主要用于基于 RPM 的发行版，如早期的 **CentOS 7、Red Hat Enterprise Linux 7** 等。

从 **CentOS 8、RHEL 8** 开始，官方已使用 **dnf** 替代 yum，但为了兼容，大部分 yum 命令仍然可以正常使用。

---

# 一、yum 的作用

yum 可以自动处理软件包之间的依赖关系，实现：

* 安装软件
* 卸载软件
* 更新软件
* 查询软件
* 管理软件仓库

相当于 Windows 中的软件管理器。

---

# 二、常用命令

## 1. 查看软件包

查询软件是否存在：

```bash
yum list nginx
```

查询已安装软件：

```bash
yum list installed
```

查询所有可安装软件：

```bash
yum list available
```

查询软件详细信息：

```bash
yum info nginx
```

例如：

```bash
yum info mysql-server
```

---

## 2. 搜索软件

根据关键字搜索：

```bash
yum search mysql
```

输出示例：

```text
mysql.x86_64
mysql-devel.x86_64
mysql-server.x86_64
```

---

## 3. 安装软件

安装单个软件：

```bash
yum install nginx
```

安装多个软件：

```bash
yum install nginx wget vim
```

自动确认：

```bash
yum install -y nginx
```

其中：

```bash
-y
```

表示自动回答 yes。

---

## 4. 卸载软件

```bash
yum remove nginx
```

自动确认：

```bash
yum remove -y nginx
```

---

## 5. 更新软件

更新指定软件：

```bash
yum update nginx
```

更新系统全部软件：

```bash
yum update
```

自动确认：

```bash
yum update -y
```

---

## 6. 查看软件来源

查看软件来自哪个仓库：

```bash
yum list nginx
```

或

```bash
yum info nginx
```

输出中会显示：

```text
Repo        : base
```

---

# 三、软件组管理

查看软件组：

```bash
yum grouplist
```

安装软件组：

```bash
yum groupinstall "Development Tools"
```

这会安装：

* gcc
* g++
* make
* gdb
* autoconf

等开发工具。

删除软件组：

```bash
yum groupremove "Development Tools"
```

---

# 四、仓库管理

查看仓库：

```bash
yum repolist
```

显示全部仓库：

```bash
yum repolist all
```

查看仓库详情：

```bash
yum repoinfo
```

---

# 五、缓存管理

yum 会缓存软件包信息。

清理缓存：

```bash
yum clean all
```

仅清理软件包缓存：

```bash
yum clean packages
```

重新生成缓存：

```bash
yum makecache
```

---

# 六、查看依赖关系

查看软件依赖：

```bash
yum deplist nginx
```

查询哪个软件提供某个文件：

```bash
yum provides /usr/bin/mysql
```

或者：

```bash
yum whatprovides libssl.so.10
```

这在解决缺少库文件问题时非常有用。

---

# 七、历史操作管理

查看历史记录：

```bash
yum history
```

查看某次安装详情：

```bash
yum history info 10
```

回滚操作：

```bash
yum history undo 10
```

---

# 八、典型运维场景

## 安装常用工具

```bash
yum install -y \
vim \
wget \
curl \
net-tools \
lrzsz \
tree
```

---

## 安装编译环境

```bash
yum groupinstall -y "Development Tools"
```

---

## 安装 Nginx

```bash
yum install -y nginx
```

启动：

```bash
systemctl start nginx
```

开机启动：

```bash
systemctl enable nginx
```

---

## 更新系统

```bash
yum clean all
yum makecache
yum update -y
```

---

# 九、常见问题

## 1. 找不到命令 yum

检查系统版本：

```bash
cat /etc/os-release
```

如果是：

* CentOS Stream 8/9
* RHEL 8/9
* Rocky Linux
* AlmaLinux

建议使用：

```bash
dnf
```

例如：

```bash
dnf install nginx
```

---

## 2. Cannot find a valid baseurl

通常是仓库配置问题。

检查：

```bash
ping www.baidu.com
```

```bash
cat /etc/yum.repos.d/*.repo
```

确认：

* 网络正常
* DNS 正常
* repo 文件配置正确

---

## 3. RPM 数据库损坏

重建数据库：

```bash
rpm --rebuilddb
```

---

# 十、yum 与 dnf 对比

| 功能    | yum             | dnf               |
| ----- | --------------- | ----------------- |
| 适用系统  | CentOS 7/RHEL 7 | CentOS 8+/RHEL 8+ |
| 依赖处理  | 较慢              | 更快                |
| 内存占用  | 较高              | 更低                |
| API支持 | 较弱              | 更完善               |
| 兼容性   | 老系统             | 新系统               |

常见命令几乎一致：

```bash
yum install nginx
```

对应：

```bash
dnf install nginx
```

对于企业环境，如果你维护的是 **CentOS 7** 服务器，重点掌握 `yum install`、`yum update`、`yum remove`、`yum search`、`yum provides`、`yum history` 这几个命令即可覆盖大部分日常运维场景。