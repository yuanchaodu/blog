---
title: Linux 安全加固
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoM_h
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/199'
---

<img src="images/Linux.svg" width="300">

# Linux 安全加固

Linux 安全加固（Linux Hardening）是指通过配置操作系统、网络、账户、服务和审计等措施，减少系统攻击面，提高系统抵御攻击能力。对于企业服务器，建议以 **CIS Benchmark（Center for Internet Security）** 或 **DISA STIG** 作为安全基线，再结合企业自身业务进行适当调整。([CIS][1])

下面按照企业生产环境介绍一套比较完整的 Linux 安全加固方案。

---

# 一、安全加固原则

Linux 安全加固遵循几个基本原则：

* 最小权限原则（Least Privilege）
* 默认拒绝（Default Deny）
* 最少服务原则
* 深度防御（Defense in Depth）
* 安全审计可追溯

安全不是安装一个软件，而是多个安全措施共同作用。

---

# 二、系统更新

这是最重要的一项。

建议：

* 安装最新安全补丁
* 定期更新系统
* 删除已经停止维护的软件

例如：

Ubuntu

```bash
apt update
apt upgrade
```

RHEL/CentOS

```bash
dnf update
```

建议企业建立：

* 每月补丁窗口
* 紧急漏洞24小时修复机制
* 测试环境验证后再升级生产环境

大量 Linux 入侵事件都利用了已公开漏洞，而及时更新是最基础也最有效的防护措施。([NOC CDN and WAF][2])

---

# 三、账户安全

## 禁止 Root 远程登录

编辑

```text
/etc/ssh/sshd_config
```

修改

```text
PermitRootLogin no
```

---

## 使用 sudo

管理员不要直接使用 root

例如

```bash
sudo su -
```

而不是：

```bash
ssh root@server
```

---

## 密码策略

要求：

* 长度≥12位
* 包含大小写
* 数字
* 特殊字符

例如：

```
Y8@kL2#Pq9!Mz
```

---

## 锁定长期不用账户

查看

```bash
lastlog
```

锁定

```bash
passwd -l username
```

---

## SSH 使用密钥登录

关闭密码登录：

```text
PasswordAuthentication no
```

开启：

```text
PubkeyAuthentication yes
```

相比密码认证，SSH 密钥能有效降低暴力破解风险。([Textbook of Linux][3])

---

# 四、SSH 安全

建议：

修改默认端口（降低扫描噪声，不能替代认证）

例如

```
22
↓

22222
```

同时：

```
MaxAuthTries 3

LoginGraceTime 30

ClientAliveInterval 300

ClientAliveCountMax 2
```

修改后：

```
systemctl restart sshd
```

---

# 五、关闭无用服务

查看：

```bash
ss -tulnp
```

或者

```bash
netstat -tunlp
```

查看服务：

```bash
systemctl list-unit-files
```

关闭：

```bash
systemctl disable telnet
systemctl stop telnet
```

企业服务器尽量只保留业务必需的端口。

---

# 六、防火墙

推荐：

Ubuntu

```
UFW
```

RHEL

```
firewalld
```

现代 Linux 也推荐直接使用 **nftables** 作为底层包过滤框架。([Andrea Farneti - Wiki][4])

原则：

```
默认拒绝

仅开放：

22
80
443
业务端口
```

例如：

```
deny all

allow ssh

allow https
```

---

# 七、SELinux / AppArmor

Linux 最重要的一层安全机制之一。

RHEL：

```
SELinux
```

Ubuntu：

```
AppArmor
```

查看 SELinux

```bash
getenforce
```

建议：

```
Enforcing
```

不要长期设置：

```
Disabled
```

强制访问控制（MAC）可以限制进程即使获得较高权限，也不能随意访问系统资源，是企业环境的重要防线。([Textbook of Linux][3])

---

# 八、文件权限

重点目录：

```
/etc

/root

/home

/var
```

查看：

```bash
find / -perm -2
```

检查：

```
777

666
```

避免：

```
chmod 777
```

敏感文件：

```
/etc/shadow

600
```

---

# 九、日志审计

建议开启：

```
rsyslog

journald

auditd
```

查看：

```
journalctl
```

安装：

```
auditd
```

记录：

* 登录
* sudo
* 文件修改
* 权限变化
* 用户创建
* 删除账户

日志建议集中保存到独立日志服务器，避免主机被攻陷后日志一并丢失。([Andrea Farneti - Wiki][4])

---

# 十、入侵防护

常见工具：

## Fail2ban

自动封禁：

```
SSH暴力破解

FTP

Apache

Nginx
```

例如：

```
5次失败

封禁30分钟
```

不过，如果 SSH 已禁用密码登录、仅允许密钥认证，Fail2ban 的收益会降低，可以结合防火墙进行连接速率限制。([Reddit][5])

---

# 十一、内核安全

修改：

```
/etc/sysctl.conf
```

例如：

关闭 IP 转发（非网关场景）

```
net.ipv4.ip_forward=0
```

开启 SYN Cookie

```
net.ipv4.tcp_syncookies=1
```

禁止 ICMP 重定向

```
net.ipv4.conf.all.accept_redirects=0
```

关闭源路由

```
net.ipv4.conf.all.accept_source_route=0
```

配置完成：

```bash
sysctl -p
```

---

# 十二、文件系统加固

建议：

```
/tmp

/var/tmp
```

增加

```
nodev

nosuid

noexec
```

例如：

```
/tmp
```

```
defaults,nodev,nosuid,noexec
```

可以降低恶意程序在临时目录执行的风险。([Ubuntu 文档][6])

---

# 十三、时间同步

保证：

```
chronyd

systemd-timesyncd
```

时间准确：

* 日志一致
* 审计准确
* Kerberos
* 数据库同步

---

# 十四、安全扫描

推荐工具：

| 工具       | 功能                 |
| -------- | ------------------ |
| Lynis    | 安全检查               |
| OpenSCAP | CIS/STIG 检查        |
| CIS-CAT  | CIS Benchmark 合规检查 |
| Nessus   | 漏洞扫描               |

建议：

每季度至少进行一次全面扫描。

---

# 十五、企业级安全体系

对于企业生产服务器，仅完成基础加固还不够，建议构建完整的纵深防御体系：

| 层次   | 建议                                 |
| ---- | ---------------------------------- |
| 身份认证 | LDAP/AD、MFA、多因素认证                  |
| 权限管理 | sudo 最小授权、RBAC                     |
| 网络安全 | VLAN、ACL、防火墙、零信任                   |
| 主机安全 | SELinux/AppArmor、EDR、文件完整性监控       |
| 漏洞管理 | 自动补丁、漏洞扫描                          |
| 日志审计 | Auditd + SIEM（如 Splunk、ELK）        |
| 备份恢复 | 离线备份、异地容灾                          |
| 基线管理 | CIS Benchmark、OpenSCAP、Ansible 自动化 |

---

# 推荐实施顺序

对于一台新的 Linux 服务器，建议按以下顺序实施安全加固：

1. 安装最新补丁。
2. 删除无用软件和关闭无用服务。
3. 配置 SSH（禁用 Root 登录、启用密钥认证）。
4. 配置防火墙，采用“默认拒绝、按需放行”策略。
5. 启用 SELinux 或 AppArmor。
6. 强化账户和密码策略。
7. 加固文件系统与内核参数（sysctl）。
8. 启用日志审计（auditd、journald）并集中保存日志。
9. 建立备份、漏洞扫描和基线检查机制（如 CIS Benchmark）。
10. 使用 Ansible、Puppet 等工具实现安全配置自动化，确保配置一致性。([Andrea Farneti - Wiki][4])

如果你的环境是 **RHEL 9 / Rocky Linux 9**、**Ubuntu 24.04 LTS**，或者用于**化工企业生产环境（MES、DCS、数据库、中间件）**，还可以进一步制定一份符合企业运维要求的 **Linux 安全加固基线（约 80～120 项检查项）**，覆盖账户、SSH、内核、文件系统、网络、防火墙、审计和合规等内容。

[1]: https://www.cisecurity.org/benchmark/red_hat_linux "CIS Red Hat Enterprise Linux Benchmarks"
[2]: https://noc.org/learn/linux-security-checklist "Basic Linux Security Checklist | NOC.org"
[3]: https://www.textbookoflinux.com/chapter/security.html "Security and Hardening — Textbook of Linux"
[4]: https://wiki.farnetiandrea.it/operating-systems/linux/deploy/linux-hardening "Linux server hardening: a practical CIS-aligned baseline for Debian / Ubuntu"
[5]: https://www.reddit.com/r/LinuxTeck/comments/1sxtq1g/linux_server_hardening_checklist_2026_simple/ "Linux Server Hardening Checklist (2026)  Simple Step-by-Step Guide"
[6]: https://documentation.ubuntu.com/aws/aws-how-to/instances/cis-hardening/ "Post-deployment CIS Hardening for Ubuntu 22.04 EC2 - Ubuntu on AWS documentation"