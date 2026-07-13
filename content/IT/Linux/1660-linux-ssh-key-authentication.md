---
title: Linux SSH 密钥认证
section: IT
category: Linux
---

# Linux SSH 密钥认证

<img src="images/Linux.svg" width="300">

SSH 密钥认证是使用一对加密密钥登录 Linux 服务器的方法，比单纯使用账号密码更安全，也更适合服务器运维和自动化任务。

## 一、基本原理

SSH 密钥由两部分组成：

* **私钥**：保存在客户端，不能泄露，相当于真正的钥匙。
* **公钥**：放在服务器上，可以公开，相当于门锁。

登录时，服务器会检查客户端是否持有与公钥匹配的私钥。私钥不会通过网络传输，服务器保存的也只是公钥。服务器通常从目标用户的 `~/.ssh/authorized_keys` 文件中读取允许登录的公钥。([OpenBSD 手册][1])

可以形象地理解为：服务器上安装了一把锁，只有持有对应钥匙的电脑才能开门。

---

## 二、配置流程

假设：

```text
客户端：管理员电脑
服务器：192.168.1.100
登录用户：admin
```

### 1. 在客户端生成密钥

推荐使用 Ed25519：

```bash
ssh-keygen -t ed25519 -a 100 -C "admin@client"
```

按照提示选择保存位置：

```text
/home/当前用户/.ssh/id_ed25519
```

生成两个文件：

```text
~/.ssh/id_ed25519       私钥
~/.ssh/id_ed25519.pub   公钥
```

OpenSSH 当前默认支持 Ed25519。它的密钥较短、计算开销较低，Ubuntu 官方文档也优先推荐这种算法。([OpenBSD 手册][2])

建议为私钥设置口令。这样即使私钥文件被复制，对方仍不能直接使用。

对于不支持 Ed25519 的老旧设备，可以使用 RSA：

```bash
ssh-keygen -t rsa -b 4096 -C "admin@client"
```

---

### 2. 将公钥复制到服务器

最简单的方法是：

```bash
ssh-copy-id admin@192.168.1.100
```

第一次执行需要输入服务器上 `admin` 用户的密码。

如果 SSH 使用非默认端口，例如 `2222`：

```bash
ssh-copy-id -p 2222 admin@192.168.1.100
```

`ssh-copy-id` 会把公钥追加到服务器用户的：

```text
/home/admin/.ssh/authorized_keys
```

文件中。([DigitalOcean][3])

---

### 3. 测试密钥登录

```bash
ssh admin@192.168.1.100
```

指定私钥时：

```bash
ssh -i ~/.ssh/id_ed25519 admin@192.168.1.100
```

使用非默认端口：

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519 admin@192.168.1.100
```

如果只要求输入私钥口令，而不再要求服务器账号密码，说明密钥认证已经生效。

---

## 三、手工安装公钥

没有 `ssh-copy-id` 时，可以手工配置。

先查看客户端公钥：

```bash
cat ~/.ssh/id_ed25519.pub
```

复制完整的一行内容，然后登录服务器执行：

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vi ~/.ssh/authorized_keys
```

将公钥粘贴到文件中，一把公钥占一行，然后设置权限：

```bash
chmod 600 ~/.ssh/authorized_keys
chmod go-w ~
```

确认文件属于目标用户：

```bash
chown -R admin:admin /home/admin/.ssh
```

在启用 SELinux 的 RHEL、Rocky Linux 或 AlmaLinux 上，还可以执行：

```bash
restorecon -Rv /home/admin/.ssh
```

---

## 四、服务器端配置

服务器配置文件通常是：

```text
/etc/ssh/sshd_config
```

也可能包含以下目录中的分段配置：

```text
/etc/ssh/sshd_config.d/
```

检查或增加：

```text
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

`AuthorizedKeysFile` 用于指定服务器从哪里读取用户的授权公钥，可以是相对于用户家目录的路径，也可以配置多个路径。([OpenBSD 手册][1])

修改后先检查语法：

```bash
sudo sshd -t
```

没有输出一般表示配置语法正确。

重新加载 SSH 服务：

Ubuntu、Debian：

```bash
sudo systemctl reload ssh
```

RHEL、Rocky Linux、AlmaLinux：

```bash
sudo systemctl reload sshd
```

查看运行状态：

```bash
sudo systemctl status sshd
```

部分 Debian、Ubuntu 系统的服务名是 `ssh`：

```bash
sudo systemctl status ssh
```

---

## 五、关闭密码登录

确认密钥登录已经成功后，可以关闭密码认证。

在服务器配置中设置：

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

同时建议禁止 root 直接登录：

```text
PermitRootLogin no
```

或者允许 root 仅通过密钥登录：

```text
PermitRootLogin prohibit-password
```

检查并重新加载：

```bash
sudo sshd -t
sudo systemctl reload sshd
```

### 重要操作顺序

不要直接关闭当前 SSH 窗口。

应当：

1. 保留当前已经登录的 SSH 会话。
2. 新开一个终端窗口。
3. 测试密钥是否可以正常登录。
4. 确认成功后，再关闭旧会话。

否则，配置错误可能导致无法远程进入服务器。

---

## 六、使用 SSH 配置文件简化登录

客户端可以编辑：

```text
~/.ssh/config
```

加入：

```text
Host production-server
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

设置权限：

```bash
chmod 600 ~/.ssh/config
```

以后只需执行：

```bash
ssh production-server
```

---

## 七、使用 ssh-agent 管理私钥

私钥设置口令后，每次连接都输入会比较麻烦，可以使用 `ssh-agent` 临时保存解锁状态。

启动代理：

```bash
eval "$(ssh-agent -s)"
```

加载私钥：

```bash
ssh-add ~/.ssh/id_ed25519
```

查看已经加载的密钥：

```bash
ssh-add -l
```

删除代理中的全部密钥：

```bash
ssh-add -D
```

`ssh-agent` 保存的是解锁后的密钥使用权限，而不是取消私钥口令。

---

## 八、常见故障排查

### 1. 仍然要求输入账号密码

使用调试模式：

```bash
ssh -vvv admin@192.168.1.100
```

重点查看是否出现：

```text
Offering public key
Server accepts key
Authenticated using publickey
```

---

### 2. 权限不正确

服务器端检查：

```bash
ls -ld /home/admin
ls -ld /home/admin/.ssh
ls -l /home/admin/.ssh/authorized_keys
```

推荐权限：

```text
/home/admin                         755 或更严格
/home/admin/.ssh                    700
/home/admin/.ssh/authorized_keys    600
```

所有者应当是 `admin` 用户，而不是 `root`：

```bash
sudo chown -R admin:admin /home/admin/.ssh
```

---

### 3. 查看服务器日志

Ubuntu、Debian：

```bash
sudo journalctl -u ssh -n 100
```

RHEL 系列：

```bash
sudo journalctl -u sshd -n 100
```

实时查看：

```bash
sudo journalctl -u sshd -f
```

也可以查看传统日志文件：

```bash
sudo tail -f /var/log/auth.log
```

或：

```bash
sudo tail -f /var/log/secure
```

---

### 4. 密钥过多导致认证失败

明确指定私钥：

```bash
ssh -o IdentitiesOnly=yes \
    -i ~/.ssh/id_ed25519 \
    admin@192.168.1.100
```

或者在 `~/.ssh/config` 中设置：

```text
IdentitiesOnly yes
```

---

## 九、安全建议

1. **私钥不要上传到服务器，也不要通过聊天软件发送。**
2. **为人工登录使用的私钥设置口令。**
3. **不同人员、不同用途使用不同密钥，避免多人共用。**
4. **离职、设备丢失或权限变化后，及时从 `authorized_keys` 删除对应公钥。**
5. **自动化账号应限制权限，不要直接使用 root。**
6. **在生产环境关闭密码认证前，先验证密钥登录并保留现有会话。**
7. **为公钥添加清晰注释，例如人员、设备和用途。**

例如：

```text
ssh-ed25519 AAAAC3... zhangsan-laptop-2026
```

标准流程可以概括为：

```text
客户端生成密钥
        ↓
公钥安装到服务器
        ↓
客户端使用私钥证明身份
        ↓
测试成功
        ↓
再关闭密码登录
```

[1]: https://man.openbsd.org/sshd_config "sshd_config(5) - OpenBSD manual pages"
[2]: https://man.openbsd.org/ssh-keygen "ssh-keygen(1) - OpenBSD manual pages"
[3]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server "How to Create an SSH Key in Linux: Easy Step-by- ..."