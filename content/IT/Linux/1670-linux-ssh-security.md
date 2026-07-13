---
title: Linux SSH 安全加固
section: IT
---

# Linux SSH 安全加固

<img src="images/Linux.svg" width="300">

SSH 是 Linux 服务器最重要的远程管理入口，也是攻击者最常扫描和尝试暴力破解的服务之一。对于企业服务器，SSH 安全加固应遵循**最小权限、密钥认证、访问控制、持续监控**四个原则。([Linuxize][1])

下面是一套适合生产环境的 SSH 安全加固方案（以 OpenSSH 为例）。

---

# 一、基础配置加固（必须）

SSH 服务配置文件：

```bash
/etc/ssh/sshd_config
```

修改配置前建议备份：

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

修改完成后先检查语法：

```bash
sshd -t
```

确认无误再重启：

```bash
systemctl restart sshd
```

整个过程中应保持已有 SSH 会话不断开，避免配置错误导致无法重新登录。([Linuxize][1])

---

# 二、禁止 Root 直接登录（★★★★★）

```text
PermitRootLogin no
```

推荐方式：

```
普通用户
    ↓
sudo
    ↓
root
```

优点：

* 记录操作日志
* 防止 Root 被暴力破解
* 满足等保及 CIS 基线要求

---

# 三、关闭密码认证（★★★★★）

采用 SSH Key 登录：

```text
PasswordAuthentication no
PubkeyAuthentication yes
```

生成密钥：

```bash
ssh-keygen -t ed25519
```

推荐使用 **Ed25519**，兼顾安全性与性能；新部署一般优先于 RSA。([Linuxize][1])

上传公钥：

```bash
ssh-copy-id user@server
```

---

# 四、限制允许登录的用户

例如：

```text
AllowUsers admin ops
```

或者

```text
AllowGroups sshusers
```

不要让所有本地账号都能 SSH 登录。

---

# 五、限制认证次数

```text
MaxAuthTries 3
```

降低暴力破解效率。

---

# 六、缩短登录等待时间

```text
LoginGraceTime 30
```

例如：

```
默认：
120 秒

修改：
30 秒
```

失败连接会更快断开。

---

# 七、禁止空密码

```text
PermitEmptyPasswords no
```

虽然默认如此，建议显式配置。

---

# 八、关闭不必要功能

如果不用，建议关闭：

```text
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
PermitTunnel no
```

减少攻击面。([Linuxize][1])

---

# 九、设置空闲超时

例如：

```text
ClientAliveInterval 300
ClientAliveCountMax 2
```

表示：

```
5分钟检测一次

连续2次无响应

自动断开
```

避免长期无人值守连接。

---

# 十、限制监听地址（可选）

例如：

```text
ListenAddress 192.168.10.5
```

如果服务器有多个网卡，可以只监听管理网络。

---

# 十一、更改默认端口（可选）

例如：

```text
Port 2222
```

说明：

**不能提升 SSH 本身的安全性**，但可以减少大量针对默认 22 端口的自动扫描和日志噪声，应与其他措施配合使用，而不是作为主要防护手段。([NOC CDN and WAF][2])

---

# 十二、启用 Fail2Ban

安装：

Ubuntu

```bash
apt install fail2ban
```

CentOS

```bash
yum install fail2ban
```

启用 SSH 防护：

```
连续失败登录

↓

自动封禁 IP

↓

一段时间后自动解除
```

如果已经完全禁用了密码登录，仅使用密钥认证，则 Fail2Ban 对抵御密码暴力破解的价值会降低，但仍可用于日志监控或其他服务。([Reddit][3])

---

# 十三、配置防火墙

例如：

iptables

```bash
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
```

或者 firewalld：

```bash
firewall-cmd
```

或者：

```
仅允许 VPN 地址登录 SSH
```

这是企业环境非常推荐的方法。

---

# 十四、SELinux/AppArmor

不要关闭。

建议：

```
Enforcing
```

而不是：

```
Disabled
```

---

# 十五、开启日志审计

日志：

```
/var/log/auth.log

或

/var/log/secure
```

重点查看：

```
Failed password

Invalid user

Accepted publickey
```

企业建议接入：

* rsyslog
* ELK
* Graylog
* Wazuh
* SIEM

进行集中审计。

---

# 十六、及时升级 OpenSSH

近几年 OpenSSH 出现过多个安全漏洞，例如：

* CVE-2025-26465（特定配置下的主机身份验证问题）
* CVE-2025-26466（预认证拒绝服务）
* 2024 年披露的 sshd 竞态条件漏洞等。

因此建议：

* 定期更新 OpenSSH
* 开启安全补丁自动更新
* 关注官方安全公告([openssh.org][4])

---

# 十七、企业级增强措施（推荐）

如果是生产环境，可以进一步采用：

| 加固措施            | 推荐程度  |
| --------------- | ----- |
| MFA（SSH + TOTP） | ★★★★★ |
| VPN 后才能 SSH     | ★★★★★ |
| 堡垒机统一登录         | ★★★★★ |
| LDAP/AD 集中认证    | ★★★★☆ |
| 跳板机（Jump Host）  | ★★★★☆ |
| 禁止公网 SSH，仅内网开放  | ★★★★★ |
| SSH CA（证书认证）    | ★★★★☆ |

---

# 推荐的 `sshd_config` 基线示例

```text
Port 22

Protocol 2

PermitRootLogin no

PasswordAuthentication no

PubkeyAuthentication yes

PermitEmptyPasswords no

MaxAuthTries 3

LoginGraceTime 30

AllowUsers admin ops

X11Forwarding no

AllowTcpForwarding no

AllowAgentForwarding no

PermitTunnel no

ClientAliveInterval 300

ClientAliveCountMax 2

UseDNS no
```

---

## 企业环境最佳实践

对于化工企业、制造业等生产网络，建议采用以下组合方案：

1. **仅允许 SSH 密钥登录（Ed25519），禁用密码认证。**
2. **禁止 Root 直接登录，通过普通账户结合 `sudo` 提权。**
3. **将 SSH 访问限制在办公网、运维网或 VPN 网络，避免直接暴露公网。**
4. **配合防火墙、Fail2Ban（或网络层限速）、日志集中审计，形成多层防护。**
5. **定期更新 OpenSSH 与系统补丁，并依据 CIS Benchmark、等保或企业安全基线开展配置检查。**([Linuxize][1])

如果你的服务器是 **Ubuntu 22.04/24.04、Rocky Linux 9、RHEL 9 或 CentOS Stream 9**，还可以根据对应系统提供一份符合 **CIS Benchmark** 的 SSH 安全基线配置。

[1]: https://linuxize.com/post/ssh-hardening-best-practices/ "SSH Hardening Guide: Best Practices for Linux Servers | Linuxize"
[2]: https://noc.org/learn/securing-ssh "10 Steps to Securing SSH on a Linux Server | NOC.org"
[3]: https://www.reddit.com/r/LinuxTeck/comments/1sxtq1g/linux_server_hardening_checklist_2026_simple/ "Linux Server Hardening Checklist (2026)  Simple Step-by-Step Guide"
[4]: https://www.openssh.org/security.html "OpenSSH: Security"