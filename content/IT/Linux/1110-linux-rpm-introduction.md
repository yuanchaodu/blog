---
title: Linux RPM 基础
section: IT
category: Linux
---

# Linux RPM 基础

<img src="images/Linux.svg" width="300">

# Linux RPM 基础

RPM（Red Hat Package Manager）是 Linux 系统中最常见的软件包管理机制之一，广泛应用于基于 Red Hat 系列的发行版，例如：

* Red Hat Enterprise Linux（RHEL）
* CentOS
* Rocky Linux
* AlmaLinux
* Fedora

RPM 本质上是一种软件打包格式，同时也是一个软件管理工具。

---

# 一、RPM是什么

可以把 RPM 理解为软件安装包，类似于 Windows 的 `.exe` 或 `.msi` 文件。

例如：

```bash
nginx-1.26.0-1.el9.x86_64.rpm
```

文件名通常包含：

```text
软件名-版本号-发布号.系统版本.架构.rpm
```

例如：

```text
nginx      软件名
1.26.0     软件版本
1          发布次数
el9        适用于RHEL9系列
x86_64     CPU架构
```

---

# 二、RPM包包含什么

一个 RPM 包通常包含：

* 程序文件
* 配置文件
* 依赖关系信息
* 安装脚本
* 卸载脚本
* 版本信息

安装 RPM 时，系统会自动将这些内容部署到指定位置。

---

# 三、查看系统已安装RPM包

查看所有安装的软件：

```bash
rpm -qa
```

查看数量：

```bash
rpm -qa | wc -l
```

例如：

```bash
rpm -qa | grep nginx
```

输出：

```bash
nginx-1.26.0-1.el9.x86_64
```

---

# 四、查询RPM包信息

## 查询软件是否安装

```bash
rpm -q nginx
```

结果：

```bash
nginx-1.26.0-1.el9.x86_64
```

未安装：

```bash
package nginx is not installed
```

---

## 查看详细信息

```bash
rpm -qi nginx
```

输出内容包括：

* 名称
* 版本
* 厂商
* 描述
* 安装日期

---

## 查看安装了哪些文件

```bash
rpm -ql nginx
```

例如：

```text
/etc/nginx/nginx.conf
/usr/sbin/nginx
/usr/share/nginx
```

---

## 查询某个文件属于哪个RPM包

例如：

```bash
rpm -qf /usr/sbin/nginx
```

结果：

```bash
nginx-1.26.0-1.el9.x86_64
```

---

# 五、安装RPM包

## 安装

```bash
rpm -ivh package.rpm
```

参数说明：

```text
-i  install 安装
-v  verbose 显示详细过程
-h  hash 显示进度条
```

示例：

```bash
rpm -ivh nginx.rpm
```

---

## 升级

```bash
rpm -Uvh nginx.rpm
```

参数：

```text
-U upgrade
```

如果旧版本存在：

```text
卸载旧版本
安装新版本
```

---

## 强制安装

```bash
rpm -ivh --force nginx.rpm
```

适用于：

* 覆盖安装
* 修复损坏安装

---

# 六、卸载RPM包

```bash
rpm -e nginx
```

或者：

```bash
rpm -e nginx-1.26.0-1.el9
```

卸载后：

```text
程序删除
配置文件可能保留
```

---

# 七、RPM依赖问题

RPM 最大的问题之一是依赖关系。

例如：

安装 nginx：

```bash
rpm -ivh nginx.rpm
```

提示：

```text
error:
libcrypto.so.3 is needed by nginx
libssl.so.3 is needed by nginx
```

说明缺少依赖包。

传统做法：

```bash
rpm -ivh openssl.rpm
rpm -ivh openssl-libs.rpm
rpm -ivh nginx.rpm
```

非常麻烦。

---

# 八、YUM/DNF与RPM的关系

现代 Linux 基本不直接使用 rpm 安装软件。

常用：

* YUM（CentOS 7）
* DNF（RHEL8/9）

关系如下：

```text
DNF/YUM
   ↓
 RPM数据库
   ↓
 RPM包
```

可以理解为：

```text
YUM/DNF = 高级管理工具
RPM = 底层管理工具
```

例如：

```bash
yum install nginx
```

或：

```bash
dnf install nginx
```

系统自动：

```text
下载软件
检查依赖
下载依赖
安装依赖
安装软件
```

---

# 九、RPM数据库

RPM 会把所有安装记录保存到数据库。

查看数据库文件：

```bash
ls /var/lib/rpm
```

常见文件：

```text
Packages
Basenames
Name
Installtid
```

RHEL8以后部分系统改为：

```text
/var/lib/rpm
```

配合 SQLite/NDB 方式管理。

---

# 十、校验RPM包

## 校验已安装软件

```bash
rpm -V nginx
```

如果无输出：

```text
表示文件未被修改
```

如果有输出：

```text
S.5....T.
```

表示：

| 标记 | 含义     |
| -- | ------ |
| S  | 文件大小变化 |
| M  | 权限变化   |
| 5  | MD5变化  |
| T  | 时间戳变化  |

---

## 验证RPM包完整性

```bash
rpm -K package.rpm
```

结果：

```text
package.rpm: digests signatures OK
```

说明软件包完整且签名有效。

---

# 十一、RPM常用命令速查表

| 功能     | 命令              |
| ------ | --------------- |
| 查询全部软件 | rpm -qa         |
| 查询软件   | rpm -q 软件名      |
| 查看详细信息 | rpm -qi 软件名     |
| 查看安装文件 | rpm -ql 软件名     |
| 查询文件归属 | rpm -qf 文件      |
| 安装     | rpm -ivh 包名.rpm |
| 升级     | rpm -Uvh 包名.rpm |
| 卸载     | rpm -e 软件名      |
| 校验软件   | rpm -V 软件名      |
| 验证RPM包 | rpm -K 包名.rpm   |

---

# 十二、企业运维中的实际应用

在企业环境（如化工企业、制造业数字化工厂）中，RPM 常用于：

### 1. 软件安装与升级

例如部署：

* Nginx
* MariaDB
* Redis

---

### 2. 安全基线检查

查看系统安装了哪些软件：

```bash
rpm -qa
```

检查是否存在：

* 未授权软件
* 高危组件
* 过期版本

---

### 3. 文件完整性检查

例如发现：

```bash
/usr/bin/sudo
```

疑似被篡改。

执行：

```bash
rpm -V sudo
```

即可验证文件是否与安装时一致。

---

### 4. 批量运维

结合 Shell 脚本：

```bash
rpm -qa > software_list.txt
```

收集服务器软件清单。

可用于：

* 等保测评
* 网络安全检查
* CMDB资产管理
* 漏洞扫描基线核对

---

# 一句话总结

RPM 是 Red Hat 系 Linux 的底层软件包管理机制，负责软件的安装、查询、升级、卸载和完整性校验；而 YUM/DNF 则是在 RPM 之上的高级管理工具，负责自动处理依赖关系和软件仓库管理。在日常运维中，通常优先使用 YUM/DNF，RPM 更多用于底层查询、故障排查和离线安装场景。