---
title: Linux groupadd 命令
section: IT
category: Linux
---

# Linux groupadd 命令

<img src="images/Linux.svg" width="300">

`groupadd` 是 Linux 中用于**创建用户组（Group）**的命令。用户组用于管理多个用户的权限，是 Linux 用户管理的重要组成部分。

## 基本语法

```bash
groupadd [选项] 组名
```

例如：

```bash
groupadd developers
```

创建一个名为 `developers` 的用户组。

---

## 常用选项

### 1. 指定组ID（GID）

```bash
groupadd -g 1001 developers
```

或

```bash
groupadd --gid 1001 developers
```

创建组时指定 GID 为 1001。

查看组信息：

```bash
grep developers /etc/group
```

输出示例：

```text
developers:x:1001:
```

---

### 2. 创建系统组

系统组通常用于运行服务程序。

```bash
groupadd -r nginx
```

或

```bash
groupadd --system nginx
```

创建系统组 `nginx`。

系统组的 GID 一般从系统预留范围内自动分配。

---

### 3. 指定 GID 范围（部分发行版支持）

```bash
groupadd -K GID_MIN=2000 -K GID_MAX=2999 project
```

在指定范围内分配 GID。

---

### 4. 非唯一 GID

允许多个组使用相同的 GID：

```bash
groupadd -g 1001 -o testgroup
```

其中：

```bash
-o
```

表示允许非唯一 GID。

---

## 查看组信息

### 查看所有组

```bash
cat /etc/group
```

或

```bash
getent group
```

---

### 查看指定组

```bash
getent group developers
```

输出：

```text
developers:x:1001:
```

---

## 给用户添加组

创建组后，可以将用户加入该组：

```bash
usermod -aG developers nick
```

说明：

* `-a`：追加
* `-G`：附加组

查看用户所属组：

```bash
groups nick
```

或

```bash
id nick
```

---

## 删除组

使用 `groupdel`：

```bash
groupdel developers
```

删除用户组。

---

## 修改组名

使用 `groupmod`：

```bash
groupmod -n dev developers
```

将组名从 `developers` 改为 `dev`。

---

## 实际应用示例

### 创建项目组

```bash
groupadd projectA
```

将多个用户加入项目组：

```bash
usermod -aG projectA user1
usermod -aG projectA user2
usermod -aG projectA user3
```

设置共享目录：

```bash
mkdir /data/projectA
chgrp projectA /data/projectA
chmod 770 /data/projectA
```

这样只有项目组成员才能访问该目录。

---

## groupadd 的工作原理

执行：

```bash
groupadd developers
```

后，系统会在：

```text
/etc/group
/etc/gshadow
```

中新增记录。

例如：

`/etc/group`

```text
developers:x:1001:
```

`/etc/gshadow`

```text
developers:!::
```

因此执行该命令通常需要 root 权限：

```bash
sudo groupadd developers
```

---

## 常见错误

### 组已存在

```bash
groupadd developers
```

报错：

```text
groupadd: group 'developers' already exists
```

检查：

```bash
getent group developers
```

---

### 权限不足

```bash
groupadd developers
```

报错：

```text
groupadd: Permission denied.
```

解决：

```bash
sudo groupadd developers
```

---

## 常见用户组管理命令对比

| 命令            | 功能       |
| ------------- | -------- |
| `groupadd`    | 创建组      |
| `groupdel`    | 删除组      |
| `groupmod`    | 修改组      |
| `groups`      | 查看用户所属组  |
| `id`          | 查看用户和组信息 |
| `usermod -aG` | 将用户加入组   |
| `gpasswd`     | 管理组成员    |

对于日常系统管理，最常见的组合是：

```bash
groupadd projectA
usermod -aG projectA user1
id user1
```

即“创建组 → 添加用户 → 验证结果”。