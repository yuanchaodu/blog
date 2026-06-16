---
title: Linux visudo 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKTw
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/76'
---

# Linux visudo 命令

<img src="images/Linux.svg" width="300">

`visudo` 是 Linux 中专门用于**安全编辑 sudo 配置文件**的命令。它的主要作用是修改 `/etc/sudoers` 文件，并在保存时自动检查语法错误，避免因配置错误导致 sudo 功能失效。

## 一、为什么不用直接编辑 sudoers 文件

虽然可以直接执行：

```bash
vi /etc/sudoers
```

但如果写错语法，可能导致：

* sudo 命令无法使用
* 普通管理员无法获取 root 权限
* 系统管理受到影响

而使用：

```bash
visudo
```

保存时会自动检查语法，例如：

```text
>>> /etc/sudoers: syntax error near line 25 <<<
What now?
```

发现错误后不会直接覆盖原文件，从而保证系统安全。

---

## 二、基本用法

### 编辑 sudoers 文件

```bash
visudo
```

默认打开：

```text
/etc/sudoers
```

---

### 指定编辑器

临时使用 vim：

```bash
EDITOR=vim visudo
```

或者：

```bash
export EDITOR=vim
visudo
```

常见编辑器：

```bash
vim
vi
nano
```

---

## 三、sudoers 文件常见配置

### 1. 允许用户执行 sudo

例如允许用户 `nick` 使用 sudo：

```text
nick ALL=(ALL) ALL
```

含义：

| 字段    | 含义        |
| ----- | --------- |
| nick  | 用户名       |
| ALL   | 所有主机      |
| (ALL) | 可以切换为任何用户 |
| ALL   | 可以执行任何命令  |

---

### 2. 用户免密码执行 sudo

```text
nick ALL=(ALL) NOPASSWD: ALL
```

执行：

```bash
sudo systemctl restart nginx
```

无需输入密码。

---

### 3. 指定命令免密码

例如：

```text
nick ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```

只能执行：

```bash
sudo systemctl restart nginx
```

其他 sudo 命令仍需密码。

---

### 4. 用户组授权

Ubuntu 中常见：

```text
%sudo ALL=(ALL:ALL) ALL
```

表示：

```bash
usermod -aG sudo nick
```

加入 sudo 组即可获得 sudo 权限。

---

CentOS/RHEL 中常见：

```text
%wheel ALL=(ALL) ALL
```

加入 wheel 组：

```bash
usermod -aG wheel nick
```

---

## 四、推荐使用 sudoers.d

现代 Linux 不建议直接修改 `/etc/sudoers`，而是通过独立配置文件管理。

先确认 sudoers 中存在：

```text
#includedir /etc/sudoers.d
```

然后创建配置：

```bash
visudo -f /etc/sudoers.d/nick
```

内容：

```text
nick ALL=(ALL) NOPASSWD: ALL
```

优点：

* 配置独立
* 不影响主 sudoers 文件
* 便于自动化运维
* 升级系统时风险小

---

## 五、语法检查

仅检查语法，不编辑：

```bash
visudo -c
```

输出示例：

```text
/etc/sudoers: parsed OK
/etc/sudoers.d/nick: parsed OK
```

---

检查指定文件：

```bash
visudo -c -f /etc/sudoers.d/nick
```

---

## 六、查看用户 sudo 权限

查看当前用户权限：

```bash
sudo -l
```

输出示例：

```text
User nick may run the following commands on localhost:
    (ALL : ALL) ALL
```

查看其他用户：

```bash
sudo -l -U nick
```

---

## 七、常见配置示例

### 运维管理员

```text
ops ALL=(ALL) ALL
```

---

### 自动化脚本账号

```text
deploy ALL=(ALL) NOPASSWD: ALL
```

---

### 仅允许重启服务

```text
monitor ALL=(root) NOPASSWD: /bin/systemctl restart nginx
```

---

### 仅允许查看日志

```text
audit ALL=(root) /usr/bin/journalctl
```

---

## 八、实际工作中的推荐做法

对于企业服务器（CentOS、RHEL、Rocky、AlmaLinux、Ubuntu）：

1. 不直接修改 `/etc/sudoers`
2. 使用：

```bash
visudo -f /etc/sudoers.d/用户名
```

3. 修改后执行：

```bash
visudo -c
```

4. 使用：

```bash
sudo -l -U 用户名
```

验证权限

这样既符合运维规范，也方便后续审计和权限管理。
