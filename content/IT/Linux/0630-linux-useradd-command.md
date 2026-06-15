---
title: Linux useradd 命令
section: IT
category: Linux
---

# Linux useradd 命令

<img src="images/Linux.svg" width="300">

`useradd` 是 Linux 中用于**创建新用户账户**的命令。它会在系统中新增用户记录，并可同时创建用户主目录、指定用户组、Shell 等属性。

## 基本语法

```bash
useradd [选项] 用户名
```

例如：

```bash
sudo useradd testuser
```

创建用户后，通常需要设置密码：

```bash
sudo passwd testuser
```

---

## 常用参数

| 参数   | 说明         |
| ---- | ---------- |
| `-m` | 创建用户主目录    |
| `-d` | 指定主目录路径    |
| `-s` | 指定登录 Shell |
| `-g` | 指定主组       |
| `-G` | 指定附加组      |
| `-u` | 指定 UID     |
| `-c` | 添加备注信息     |
| `-e` | 设置账户失效日期   |
| `-r` | 创建系统账户     |
| `-M` | 不创建主目录     |

---

## 常见示例

### 1. 创建普通用户并自动创建家目录

```bash
sudo useradd -m tom
sudo passwd tom
```

结果：

```text
/home/tom
```

目录会自动创建。

---

### 2. 指定用户 UID

```bash
sudo useradd -u 2001 tom
```

查看：

```bash
id tom
```

输出类似：

```text
uid=2001(tom) gid=2001(tom)
```

---

### 3. 指定登录 Shell

例如使用 Bash：

```bash
sudo useradd -m -s /bin/bash tom
```

或者禁止登录：

```bash
sudo useradd -s /sbin/nologin ftpuser
```

常用于服务账号。

---

### 4. 指定主组

先创建组：

```bash
sudo groupadd dev
```

创建用户：

```bash
sudo useradd -m -g dev tom
```

查看：

```bash
id tom
```

输出：

```text
uid=1001(tom) gid=1002(dev)
```

---

### 5. 添加附加组

例如让用户拥有 sudo 权限：

```bash
sudo useradd -m -G wheel tom
```

在 Ubuntu 中：

```bash
sudo useradd -m -G sudo tom
```

查看：

```bash
groups tom
```

---

### 6. 指定用户主目录

```bash
sudo useradd -d /data/tom -m tom
```

结果：

```text
/data/tom
```

成为用户家目录。

---

### 7. 创建系统用户

```bash
sudo useradd -r nginx
```

特点：

* UID 较小（通常 <1000）
* 不允许普通登录
* 用于运行服务进程

例如：

```bash
nginx
mysql
redis
```

等服务账户。

---

## 查看用户信息

### 查看用户 UID 和组

```bash
id tom
```

### 查看用户记录

```bash
grep tom /etc/passwd
```

示例：

```text
tom:x:1001:1001::/home/tom:/bin/bash
```

字段含义：

```text
用户名:密码占位:UID:GID:备注:家目录:Shell
```

---

## 修改用户

虽然 `useradd` 用于创建用户，但修改用户通常使用 `usermod`：

修改 Shell：

```bash
sudo usermod -s /bin/zsh tom
```

添加附加组：

```bash
sudo usermod -aG docker tom
```

> 注意：`-aG` 必须一起使用，否则可能覆盖原有附加组。

---

## 删除用户

删除用户：

```bash
sudo userdel tom
```

删除用户及家目录：

```bash
sudo userdel -r tom
```

---

## 企业运维中的典型用法

创建应用运行账号：

```bash
sudo useradd -r -s /sbin/nologin appuser
```

创建开发人员账号：

```bash
sudo useradd -m -s /bin/bash -G sudo,docker dev01
sudo passwd dev01
```

创建数据库运维账号：

```bash
sudo useradd -m -g dba -G wheel dbadmin
```

---

## useradd 与 adduser 的区别

| 项目      | useradd | adduser |
| ------- | ------- | ------- |
| 类型      | 底层命令    | 友好脚本    |
| 交互性     | 无       | 有       |
| 自动创建家目录 | 视配置而定   | 默认创建    |
| 适合      | 脚本自动化   | 手工创建用户  |

例如：

```bash
adduser tom
```

会自动提示输入：

* 密码
* 姓名
* 电话
* 邮箱等信息

而：

```bash
useradd tom
```

只创建账户，不会询问任何信息。

在自动化运维脚本、Ansible、Shell 脚本中，通常优先使用 `useradd`；在日常人工管理用户时，很多发行版更常使用 `adduser`。