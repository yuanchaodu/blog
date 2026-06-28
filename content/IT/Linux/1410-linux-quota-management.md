---
title: Linux 磁盘配额
section: IT
category: Linux
---

# Linux 磁盘配额

<img src="images/Linux.svg" width="300">

Linux 磁盘配额（Disk Quota）是一种限制用户或用户组磁盘使用量的机制。它可以防止个别用户占满整个文件系统，广泛用于服务器、共享存储、高校机房和企业环境。

---

# 一、什么是磁盘配额

简单来说，磁盘配额就是给每个用户（User）或用户组（Group）规定：

* 最多可以占用多少磁盘空间（Space）
* 最多可以创建多少文件（Inode）

例如：

| 用户         | 最大空间  | 最大文件数   |
| ---------- | ----- | ------- |
| user1      | 10GB  | 100000  |
| user2      | 50GB  | 500000  |
| developer组 | 500GB | 5000000 |

当超过限制时，就不能继续写入数据。

可以把它理解成银行账户额度：

* 磁盘空间 = 存款金额
* inode = 能开的账户数量（文件数量）

即使还有很多空间，如果 inode 用完，也无法再创建新文件。

---

# 二、Linux 配额类型

Linux 支持三种配额。

## 1. 用户配额（User Quota）

限制某个用户。

例如：

```
user1
    最大10GB

user2
    最大20GB
```

最常见。

---

## 2. 用户组配额（Group Quota）

限制整个组。

例如：

```
design组

总共100GB
```

所有成员共同使用。

---

## 3. 项目配额（Project Quota）

Project Quota 不是针对用户，而是针对目录。

例如：

```
/data/projectA

限制500GB
```

无论谁写进去，都算 ProjectA 的配额。

在 XFS 文件系统中应用非常广泛。

---

# 三、限制两种资源

Linux 实际限制的是两种资源。

## （1）磁盘容量 Block

例如：

```
100GB
```

表示最多使用多少磁盘。

---

## （2）inode 数量

inode 保存文件元数据。

例如：

```
最多100万个文件
```

如果：

```
inode = 100%
```

即使还有 5TB 空间，也不能创建文件。

很多日志服务器都会碰到这种情况。

---

# 四、软限制（Soft Limit）和硬限制（Hard Limit）

Linux 配额不是只有一个限制。

而是：

```
Soft Limit
Hard Limit
```

例如：

```
Soft = 90GB

Hard = 100GB
```

过程如下：

```
80GB
 ↓
90GB（超过Soft）
 ↓
收到警告

还能继续写

↓

100GB

立即禁止写入
```

---

## Grace Period（宽限期）

超过 Soft 后不会马上停止。

例如：

```
Soft = 90GB

Hard = 100GB

Grace = 7天
```

表示：

```
今天达到95GB

还能继续用

7天后

如果仍超过90GB

禁止写入
```

因此：

```
Soft = 提醒

Hard = 强制
```

---

# 五、Linux 配额工作流程

```
写文件

     │
     ▼

Kernel

     │

检查quota

     │

是否超过限制？

 ┌──────────┐
 │ 没超过    │
 └──────────┘
      │
      ▼
允许写入

──────────────

 ┌──────────┐
 │ 超过Hard │
 └──────────┘
      │
      ▼
拒绝写入
```

---

# 六、配置流程（以 ext4 为例）

## 第一步：安装 quota 软件

不同发行版命令不同，例如：

**Debian/Ubuntu：**

```bash
sudo apt install quota
```

**RHEL/CentOS/Rocky/AlmaLinux：**

```bash
sudo dnf install quota
```

---

## 第二步：修改 `/etc/fstab`

例如：

```
UUID=xxxx /home ext4 defaults,usrquota,grpquota 0 2
```

增加：

```
usrquota
grpquota
```

例如：

```
UUID=xxxx

/home

ext4

defaults,usrquota,grpquota
```

---

## 第三步：重新挂载

```bash
mount -o remount /home
```

或者重启。

---

## 第四步：创建 quota 数据库

```bash
quotacheck -cug /home
```

参数：

```
c
创建

u
用户

g
组
```

生成：

```
aquota.user
aquota.group
```

---

## 第五步：开启 quota

```bash
quotaon /home
```

查看：

```bash
quotaon -p /home
```

---

# 七、配置用户配额

例如：

```bash
edquota user1
```

会打开编辑器：

```
Disk quotas for user user1

Filesystem

blocks

soft

hard

inodes

soft

hard

/dev/sda2

0

10240000

10485760

0

100000

120000
```

意思：

```
Soft

10GB

Hard

10.5GB

inode

100000

120000
```

保存即可生效。

---

# 八、复制配额

多个用户一样时：

```bash
edquota -p user1 user2
```

表示：

```
把 user1 的配置

复制给

user2
```

---

# 九、查看配额

查看自己的配额：

```bash
quota
```

查看指定用户：

```bash
quota user1
```

查看整个系统：

```bash
repquota /home
```

示例：

```
*** Report for /home ***

User

used

soft

hard

user1

8G

10G

12G

user2

15G

20G

25G
```

---

# 十、关闭配额

```bash
quotaoff /home
```

重新开启：

```bash
quotaon /home
```

---

# 十一、常用命令汇总

| 命令           | 作用           |
| ------------ | ------------ |
| `quota`      | 查看当前用户配额     |
| `repquota`   | 查看文件系统所有用户配额 |
| `quotaon`    | 开启配额         |
| `quotaoff`   | 关闭配额         |
| `quotacheck` | 检查并创建配额数据库   |
| `edquota`    | 编辑用户/组配额     |
| `setquota`   | 通过命令行直接设置配额  |
| `warnquota`  | 向超额用户发送警告邮件  |

其中，`setquota` 适合脚本自动化，例如：

```bash
setquota -u user1 1048576 2097152 10000 12000 /home
```

含义如下：

* `-u user1`：为用户 `user1` 设置配额。
* `1048576`：块软限制（通常以 KB 为单位，约 1 GB）。
* `2097152`：块硬限制（约 2 GB）。
* `10000`：inode 软限制。
* `12000`：inode 硬限制。
* `/home`：目标文件系统。

---

# 十二、企业中的典型应用

| 场景        | 配额对象  | 说明                |
| --------- | ----- | ----------------- |
| 高校实验室     | 用户    | 防止学生占满服务器         |
| FTP 文件服务器 | 用户    | 控制上传空间            |
| 企业 NAS    | 用户/组  | 为部门分配存储容量         |
| 开发服务器     | 用户    | 防止日志、临时文件无限增长     |
| HPC 集群    | 用户    | 限制计算任务产生的数据       |
| 容器平台      | 项目/目录 | 限制持久化卷（PV）或项目目录容量 |

## XFS 文件系统中的项目配额

在企业中，如果使用 **XFS** 文件系统，通常更推荐使用 **Project Quota**。它以目录为单位进行限制，不依赖文件所有者，因此非常适合共享目录、应用数据目录以及容器持久化存储等场景。相比传统的用户配额，它更容易实现“某个目录最多只能使用多少空间”的管理需求。

---

## 总结

Linux 磁盘配额的核心目标是控制磁盘资源的公平使用。它可以针对**用户、用户组或目录（项目）**进行限制，同时分别控制**磁盘空间（Blocks）**和**文件数量（Inodes）**。通过**软限制（Soft Limit）**、**硬限制（Hard Limit）**和**宽限期（Grace Period）**，管理员既能给用户预警，也能在达到上限时强制阻止继续写入。掌握 `quotacheck`、`quotaon`、`edquota`、`setquota` 和 `repquota` 等命令后，就可以完成绝大多数 Linux 磁盘配额的部署和日常管理工作。