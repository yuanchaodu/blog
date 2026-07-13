---
title: Linux SSH 基础
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4Anv2h
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/165'
---

# Linux SSH 基础

<img src="images/Linux.svg" width="300">

Linux SSH（Secure Shell，安全外壳协议）是 Linux 系统中最常用的远程登录和远程管理工具。它可以通过加密连接，安全地访问远程服务器，也可以用于文件传输、端口转发和自动化运维。

下面从基础概念到常用操作进行介绍。

---

# 一、什么是 SSH？

SSH（Secure Shell）是一种网络协议，用于：

* 远程登录 Linux
* 执行远程命令
* 安全传输文件
* 建立安全隧道（端口转发）

SSH 默认使用：

> TCP 22 端口

例如：

```
你的电脑  ─────SSH─────> Linux服务器
```

整个通信过程都是加密的，比 Telnet 安全得多。

---

# 二、SSH 的组成

SSH 通常包括两个部分：

## 1）SSH Client（客户端）

就是发起连接的一方。

Linux 自带：

```
ssh
scp
sftp
ssh-keygen
ssh-copy-id
```

Windows 可以使用：

* OpenSSH（Windows 已内置）
* PuTTY
* MobaXterm
* Xshell

---

## 2）SSH Server（服务端）

Linux 上运行：

```
sshd
```

它负责：

* 接收连接
* 用户认证
* 建立加密通道
* 执行命令

Ubuntu 安装：

```bash
sudo apt install openssh-server
```

CentOS：

```bash
sudo yum install openssh-server
```

启动：

```bash
sudo systemctl start sshd
```

查看状态：

```bash
systemctl status sshd
```

设置开机启动：

```bash
sudo systemctl enable sshd
```

---

# 三、SSH 登录

最基本命令：

```bash
ssh 用户名@IP地址
```

例如：

```bash
ssh root@192.168.1.100
```

第一次连接会看到：

```
The authenticity of host can't be established.
```

输入：

```
yes
```

之后输入密码即可。

---

如果服务器端口不是22：

```bash
ssh -p 2222 root@192.168.1.100
```

---

# 四、SSH 登录流程

可以理解为下面几个步骤：

```
客户端
    │
    │ 建立TCP连接
    ▼
服务器22端口
    │
    │ 协商加密算法
    ▼
交换密钥
    │
    │ 身份认证
    ▼
建立加密通道
    │
    ▼
Shell
```

整个过程的数据都会被加密。

---

# 五、密码登录

最简单方式：

```
ssh root@server
```

输入：

```
Password:
```

验证成功即可登录。

优点：

* 简单

缺点：

* 每次输入密码
* 容易被暴力破解
* 不适合自动化

---

# 六、公钥认证（推荐）

SSH 更推荐使用密钥对认证。

原理：

```
        私钥（自己保存）
             │
             │
             ▼
      SSH Client
             │
============互联网============
             │
             ▼
      SSH Server
             │
      公钥（服务器保存）
```

服务器保存：

```
~/.ssh/authorized_keys
```

客户端保存：

```
~/.ssh/id_rsa
```

或：

```
~/.ssh/id_ed25519
```

---

## 生成密钥

推荐使用 Ed25519 算法：

```bash
ssh-keygen -t ed25519
```

也可以使用 RSA：

```bash
ssh-keygen -t rsa -b 4096
```

一路回车即可。

生成：

```
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

其中：

```
id_ed25519
```

是私钥。

```
id_ed25519.pub
```

是公钥。

---

## 上传公钥

推荐：

```bash
ssh-copy-id user@server
```

或者：

```bash
cat id_ed25519.pub >> ~/.ssh/authorized_keys
```

之后即可：

```
ssh user@server
```

无需密码。

---

# 七、SSH 配置文件

客户端配置：

```
~/.ssh/config
```

例如：

```text
Host web
    HostName 192.168.1.100
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

以后：

```bash
ssh web
```

即可连接。

---

# 八、SSH 常用参数

## 指定端口

```bash
ssh -p 2222 user@host
```

---

## 指定密钥

```bash
ssh -i ~/.ssh/id_ed25519 user@host
```

---

## 执行远程命令

```bash
ssh user@host "ls -l"
```

结果直接返回本地。

例如：

```bash
ssh root@192.168.1.100 "df -h"
```

---

## 后台运行

```bash
ssh -f
```

一般配合端口转发使用。

---

## 输出调试信息

```bash
ssh -v
```

更详细：

```bash
ssh -vv
```

最详细：

```bash
ssh -vvv
```

---

# 九、SSH 文件传输

## SCP

上传：

```bash
scp file.txt user@host:/home/user/
```

下载：

```bash
scp user@host:/tmp/file.txt .
```

复制目录：

```bash
scp -r dir user@host:/tmp/
```

---

## SFTP

连接：

```bash
sftp user@host
```

进入：

```
sftp>
```

上传：

```
put test.txt
```

下载：

```
get test.txt
```

退出：

```
exit
```

---

# 十、SSH 免密登录

流程：

```
生成密钥

↓

上传公钥

↓

服务器保存公钥

↓

客户端证明拥有私钥

↓

登录成功
```

整个过程无需传输私钥，因此安全性较高。

---

# 十一、SSH 服务配置

服务器配置文件：

```
/etc/ssh/sshd_config
```

常见配置：

允许 Root 登录：

```text
PermitRootLogin yes
```

禁止密码登录（启用密钥认证后推荐）：

```text
PasswordAuthentication no
```

修改端口：

```text
Port 2222
```

禁止空密码：

```text
PermitEmptyPasswords no
```

修改后重启：

```bash
sudo systemctl restart sshd
```

---

# 十二、查看 SSH 状态

查看服务：

```bash
systemctl status sshd
```

查看监听端口：

```bash
ss -tlnp | grep ssh
```

查看日志：

基于 `systemd` 的发行版：

```bash
journalctl -u sshd
```

Ubuntu/Debian（传统日志）：

```bash
sudo tail -f /var/log/auth.log
```

CentOS/RHEL：

```bash
sudo tail -f /var/log/secure
```

---

# 十三、常见问题

| 问题                           | 原因              | 解决方法                                 |
| ---------------------------- | --------------- | ------------------------------------ |
| Connection refused           | SSH 服务未启动或端口未监听 | 启动 `sshd`，检查端口和防火墙                   |
| Permission denied            | 用户名、密码或密钥错误     | 检查账户、密钥及 `~/.ssh` 权限                 |
| Connection timed out         | 网络不通、防火墙阻止      | 检查网络连通性和安全组配置                        |
| Host key verification failed | 主机密钥变化          | 删除本地 `~/.ssh/known_hosts` 中对应条目后重新连接 |

---

# 十四、SSH 安全建议

在生产环境中，建议采取以下措施：

* 使用 **Ed25519** 密钥认证，避免长期使用密码登录。
* 登录后及时禁用密码认证（`PasswordAuthentication no`），仅保留密钥登录。
* 尽量避免直接使用 `root` 登录，可先使用普通用户登录，再通过 `sudo` 提权。
* 修改默认 22 端口可以减少自动扫描，但不能替代其他安全措施。
* 配置防火墙，仅允许可信 IP 访问 SSH 端口。
* 定期更新 OpenSSH 和操作系统，修复安全漏洞。
* 开启详细日志并定期检查异常登录行为，必要时配合失败登录限制工具（如 Fail2Ban）。
