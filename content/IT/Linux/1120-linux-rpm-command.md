---
title: Linux RPM 命令
section: IT
category: Linux
---

# Linux RPM 命令

<img src="images/Linux.svg" width="300">

`rpm`（RPM Package Manager）是 Linux 系统中常见的软件包管理工具，主要用于 **Red Hat、CentOS、Rocky Linux、AlmaLinux、Oracle Linux、openSUSE** 等发行版的软件包管理。

需要注意的是，`rpm` 只负责软件包的安装、卸载和查询，本身**不自动解决依赖关系**。实际生产环境中通常配合 `yum` 或 `dnf` 使用。

## 一、RPM 常用命令

### 1. 安装软件包

```bash
rpm -ivh package.rpm
```

参数说明：

* `-i`：安装（install）
* `-v`：显示详细信息（verbose）
* `-h`：显示安装进度（hash）

例如：

```bash
rpm -ivh nginx-1.26.0.rpm
```

---

### 2. 升级软件包

```bash
rpm -Uvh package.rpm
```

例如：

```bash
rpm -Uvh nginx-1.27.0.rpm
```

特点：

* 已安装则升级
* 未安装则直接安装

---

### 3. 强制升级

```bash
rpm -Uvh --force package.rpm
```

适用于：

* 版本降级
* 文件冲突处理

---

### 4. 卸载软件包

```bash
rpm -e 软件包名
```

例如：

```bash
rpm -e nginx
```

查看软件包名称：

```bash
rpm -qa | grep nginx
```

---

### 5. 强制卸载（不检查依赖）

```bash
rpm -e --nodeps nginx
```

生产环境慎用。

---

## 二、查询软件包

### 1. 查询所有已安装软件

```bash
rpm -qa
```

例如：

```bash
rpm -qa | less
```

---

### 2. 查询指定软件是否安装

```bash
rpm -q nginx
```

输出：

```text
nginx-1.26.0-1.el9.x86_64
```

未安装：

```text
package nginx is not installed
```

---

### 3. 查看软件详细信息

```bash
rpm -qi nginx
```

显示：

* 版本
* 厂商
* 安装日期
* 描述

等信息。

---

### 4. 查看软件安装了哪些文件

```bash
rpm -ql nginx
```

例如：

```text
/etc/nginx/nginx.conf
/usr/sbin/nginx
/usr/lib/systemd/system/nginx.service
```

---

### 5. 查询某个文件属于哪个软件包

```bash
rpm -qf 文件路径
```

例如：

```bash
rpm -qf /usr/sbin/nginx
```

输出：

```text
nginx-1.26.0-1.el9.x86_64
```

---

### 6. 查看软件配置文件

```bash
rpm -qc nginx
```

例如：

```text
/etc/nginx/nginx.conf
/etc/nginx/mime.types
```

---

### 7. 查看帮助文档

```bash
rpm -qd nginx
```

---

## 三、查询 RPM 包信息（未安装）

假设当前目录有：

```text
nginx-1.26.0.rpm
```

### 查看软件信息

```bash
rpm -qpi nginx-1.26.0.rpm
```

---

### 查看包含文件

```bash
rpm -qpl nginx-1.26.0.rpm
```

---

### 查看依赖关系

```bash
rpm -qpR nginx-1.26.0.rpm
```

输出示例：

```text
/bin/sh
libc.so.6
libpcre.so.1
systemd
```

---

## 四、验证与校验

### 1. 验证软件完整性

```bash
rpm -V nginx
```

如果无输出：

```text
（表示文件未被修改）
```

如果有输出：

```text
S.5....T. /etc/nginx/nginx.conf
```

表示文件属性发生变化。

常见标记：

| 标记 | 含义     |
| -- | ------ |
| S  | 文件大小变化 |
| M  | 权限变化   |
| 5  | MD5变化  |
| T  | 时间戳变化  |
| U  | 所有者变化  |
| G  | 所属组变化  |

---

### 2. 校验 RPM 包

```bash
rpm -K nginx.rpm
```

或者

```bash
rpm --checksig nginx.rpm
```

输出：

```text
nginx.rpm: digests signatures OK
```

---

## 五、重建 RPM 数据库

RPM 数据库损坏时可使用：

### 查看数据库文件

```bash
ls /var/lib/rpm
```

---

### 重建数据库

```bash
rpm --rebuilddb
```

或者：

```bash
rpm --initdb
```

---

## 六、生产环境常用组合

### 查找已安装软件

```bash
rpm -qa | grep mysql
```

---

### 查看软件版本

```bash
rpm -qi mysql-community-server
```

---

### 查询命令属于哪个包

```bash
rpm -qf $(which mysql)
```

---

### 查看某个 RPM 包依赖

```bash
rpm -qpR mysql.rpm
```

---

### 查看软件安装路径

```bash
rpm -ql mysql-community-server
```

---

## 七、RPM 与 YUM/DNF 的区别

| 功能     | RPM | YUM/DNF |
| ------ | --- | ------- |
| 安装软件   | √   | √       |
| 升级软件   | √   | √       |
| 卸载软件   | √   | √       |
| 查询软件   | √   | √       |
| 自动解决依赖 | ×   | √       |
| 在线仓库管理 | ×   | √       |
| 批量更新   | ×   | √       |

例如：

```bash
dnf install nginx
```

或：

```bash
yum install nginx
```

会自动下载并安装依赖，而：

```bash
rpm -ivh nginx.rpm
```

若缺少依赖则会直接报错。

## 八、运维人员最常用的 RPM 命令速查

```bash
# 查看是否安装
rpm -q nginx

# 查看详细信息
rpm -qi nginx

# 查看安装文件
rpm -ql nginx

# 查询文件属于哪个包
rpm -qf /usr/sbin/nginx

# 查看所有软件
rpm -qa

# 安装
rpm -ivh package.rpm

# 升级
rpm -Uvh package.rpm

# 卸载
rpm -e nginx

# 查看RPM包依赖
rpm -qpR package.rpm

# 验证软件完整性
rpm -V nginx

# 重建RPM数据库
rpm --rebuilddb
```

对于企业服务器日常运维，最常用的其实是：

* `rpm -qa`
* `rpm -qi`
* `rpm -ql`
* `rpm -qf`
* `rpm -ivh`
* `rpm -Uvh`
* `rpm -e`
* `rpm -V`

这几个命令基本覆盖了软件安装、查询、升级、卸载和故障排查的大部分场景。