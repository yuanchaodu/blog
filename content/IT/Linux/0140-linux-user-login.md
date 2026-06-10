---
title: Linux 用户登录机制
section: IT
category: Linux
---

# Linux 用户登录机制

<img src="images/Linux.svg" width="300">

Linux 的用户登录机制，本质上是一个“身份认证 + 会话创建 + 权限控制”的过程。用户从输入用户名和密码开始，到获得 Shell 操作环境结束，期间涉及多个系统组件协同工作。

## 一、登录过程总体流程

```text
用户输入用户名和密码
        ↓
登录程序（login、sshd、gdm等）
        ↓
PAM认证模块
        ↓
验证用户信息
        ↓
读取用户配置
        ↓
创建用户会话
        ↓
启动Shell
        ↓
获得系统操作权限
```

可以把 Linux 看成一个办公楼：

* 用户账号 = 工牌
* 密码 = 身份验证
* PAM = 门禁系统
* Shell = 分配给你的办公室
* 文件权限 = 你能进入哪些房间

---

## 二、用户信息存储

Linux 用户信息主要保存在以下文件中：

### 1. /etc/passwd

存放用户基本信息。

示例：

```text
nick:x:1001:1001:Nick:/home/nick:/bin/bash
```

字段含义：

```text
用户名
密码占位符
UID
GID
描述信息
家目录
登录Shell
```

例如：

```text
nick:x:1001:1001:Nick:/home/nick:/bin/bash
```

表示：

* 用户名：nick
* UID：1001
* 主组ID：1001
* 家目录：/home/nick
* 默认Shell：bash

---

### 2. /etc/shadow

存放加密后的密码。

例如：

```text
nick:$6$abc123...:19800:0:99999:7:::
```

只有 root 可以读取：

```bash
ls -l /etc/shadow
```

通常显示：

```text
-r-------- root root
```

这是 Linux 安全设计的重要部分。

---

## 三、登录认证过程

### 本地终端登录

例如：

```text
login:
```

输入：

```text
Username: nick
Password: ******
```

系统执行：

1. 查找 `/etc/passwd`
2. 获取用户UID
3. 查找 `/etc/shadow`
4. 获取密码哈希
5. 对输入密码进行哈希计算
6. 比较两个哈希值

如果一致：

```text
认证成功
```

否则：

```text
Login incorrect
```

---

## 四、PAM认证机制

现代 Linux 几乎都使用 PAM。

PAM 全称：

**Pluggable Authentication Modules**

即：

> 可插拔认证模块

对应目录：

```bash
/etc/pam.d/
```

例如：

```bash
/etc/pam.d/login
/etc/pam.d/sshd
```

登录时：

```text
login
  ↓
PAM
  ↓
认证模块
```

PAM 可以支持：

* 本地密码认证
* LDAP认证
* Active Directory认证
* 双因素认证（MFA）
* USB Key认证
* 指纹认证

例如：

```text
sshd
 ↓
PAM
 ↓
Google Authenticator
 ↓
密码认证
 ↓
允许登录
```

企业环境大量依赖 PAM 实现统一认证。

---

## 五、用户会话创建

认证成功后，系统开始创建会话。

主要步骤：

### 1. 设置 UID 和 GID

例如：

```text
UID=1001
GID=1001
```

查看：

```bash
id
```

输出：

```text
uid=1001(nick)
gid=1001(nick)
groups=1001(nick),10(wheel)
```

---

### 2. 切换到用户家目录

例如：

```bash
cd /home/nick
```

---

### 3. 设置环境变量

例如：

```bash
HOME=/home/nick
USER=nick
PATH=/usr/bin:/bin
```

查看：

```bash
env
```

---

### 4. 记录登录信息

保存到：

```bash
/var/run/utmp
/var/log/wtmp
```

查看当前登录：

```bash
who
```

查看历史登录：

```bash
last
```

---

## 六、启动用户Shell

Linux 最终会启动用户配置的 Shell。

例如：

```bash
/bin/bash
```

或者：

```bash
/bin/zsh
/bin/sh
```

查看：

```bash
echo $SHELL
```

输出：

```text
/bin/bash
```

登录完成后：

```text
nick@server:~$
```

此时用户已经获得交互环境。

---

## 七、SSH登录机制

企业服务器最常见的登录方式。

### SSH登录流程

```text
SSH Client
      ↓
TCP 22
      ↓
sshd
      ↓
PAM认证
      ↓
创建会话
      ↓
启动Shell
```

认证方式有两种。

### 密码认证

```bash
ssh nick@server
```

输入密码。

---

### 密钥认证

客户端生成：

```bash
ssh-keygen
```

生成：

```text
id_rsa
id_rsa.pub
```

公钥放到服务器：

```bash
~/.ssh/authorized_keys
```

登录时：

```text
客户端证明拥有私钥
↓
服务器验证公钥
↓
登录成功
```

企业生产环境通常禁用密码登录，只允许密钥认证。

---

## 八、权限控制机制

登录成功后，Linux 根据 UID 和 GID 判断权限。

例如文件：

```bash
-rw-r-----
```

所有者：

```text
nick
```

所属组：

```text
itgroup
```

判断顺序：

```text
是否为Owner
    ↓
是否属于Group
    ↓
Others
```

对应权限：

```text
r = 读
w = 写
x = 执行
```

---

## 九、登录相关日志

### 认证日志

Ubuntu：

```bash
/var/log/auth.log
```

RHEL/CentOS：

```bash
/var/log/secure
```

查看：

```bash
sudo tail -f /var/log/secure
```

示例：

```text
Accepted password for nick
Accepted publickey for nick
Failed password for root
```

---

## 十、企业环境中的典型登录架构

现代企业通常不是本地账号认证，而是集中身份管理。

```text
用户
 ↓
SSH
 ↓
Linux服务器
 ↓
PAM
 ↓
LDAP/AD
 ↓
统一身份认证
```

常见方案：

* OpenLDAP
* FreeIPA
* Microsoft Active Directory
* SSSD
* Kerberos 单点登录（SSO）
* MFA 多因素认证

这样可以实现：

* 统一账号管理
* 统一密码策略
* 统一权限控制
* 审计追踪

## 总结

Linux 用户登录机制可以概括为：

```text
登录程序
   ↓
PAM认证
   ↓
验证用户身份
   ↓
设置UID/GID
   ↓
创建用户会话
   ↓
加载环境变量
   ↓
启动Shell
   ↓
获得系统权限
```

从系统实现角度看，最核心的四个组件是：

1. `/etc/passwd` —— 用户信息
2. `/etc/shadow` —— 密码信息
3. PAM —— 身份认证框架
4. Shell —— 用户交互环境

理解这四部分后，再深入学习 SSH、LDAP、AD、Kerberos 和 Linux 权限模型，就能掌握企业级 Linux 身份认证与访问控制体系。