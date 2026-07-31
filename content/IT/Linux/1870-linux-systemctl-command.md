---
title: Linux systemctl 命令
section: IT
category: Linux
---

<img src="images/Linux.svg" width="300">

# Linux systemctl 命令

`systemctl` 是 Linux 中用于管理 **systemd** 系统和服务的命令。它是目前大多数主流 Linux 发行版（如 Ubuntu、CentOS 7+、Rocky Linux、AlmaLinux、Debian、Fedora 等）的标准服务管理工具，用来替代早期的 `service` 和 `chkconfig` 命令。

## 一、通俗理解

可以把 `systemctl` 理解为**服务器的总控制台**。

它可以管理：

* 系统服务（如 nginx、mysql、httpd）
* 系统启动项
* 系统运行状态
* 日志（配合 `journalctl`）
* 电源管理（重启、关机等）

例如：

> Windows 有"服务（Services）"管理器，而 Linux 中 `systemctl` 就是管理这些服务的命令。

---

## 二、常用命令

### 1. 查看服务状态

```bash
systemctl status 服务名
```

例如：

```bash
systemctl status nginx
```

输出示例：

```text
● nginx.service - The nginx HTTP Server
   Loaded: loaded
   Active: active (running)
```

其中：

* **Loaded**：是否已安装
* **Active**：是否正在运行
* **Main PID**：主进程号

---

### 2. 启动服务

```bash
systemctl start 服务名
```

例如：

```bash
systemctl start nginx
```

仅本次启动。

---

### 3. 停止服务

```bash
systemctl stop nginx
```

停止服务。

---

### 4. 重启服务

```bash
systemctl restart nginx
```

适用于修改配置之后。

---

### 5. 重新加载配置（不中断服务）

```bash
systemctl reload nginx
```

很多服务（如 Nginx）支持热加载。

如果不确定：

```bash
systemctl reload-or-restart nginx
```

---

### 6. 查看是否运行

```bash
systemctl is-active nginx
```

结果：

```text
active
```

或者

```text
inactive
```

---

### 7. 查看是否开机启动

```bash
systemctl is-enabled nginx
```

输出：

```text
enabled
```

或者：

```text
disabled
```

---

## 三、开机启动管理

### 设置开机启动

```bash
systemctl enable nginx
```

作用：

> 下次系统启动时自动启动 nginx。

---

### 取消开机启动

```bash
systemctl disable nginx
```

---

### 启动并设置开机启动

```bash
systemctl enable --now nginx
```

相当于：

```bash
systemctl enable nginx
systemctl start nginx
```

---

### 停止并取消开机启动

```bash
systemctl disable --now nginx
```

---

## 四、查看所有服务

### 查看运行中的服务

```bash
systemctl list-units --type=service
```

---

### 查看全部服务

```bash
systemctl list-unit-files --type=service
```

例如：

```text
nginx.service          enabled
sshd.service           enabled
firewalld.service      disabled
```

---

## 五、查看失败的服务

```bash
systemctl --failed
```

例如：

```text
UNIT              LOAD   ACTIVE SUB DESCRIPTION
mysql.service     loaded failed failed MySQL Server
```

这是排查系统问题时常用的命令。

---

## 六、查看服务配置

查看 Unit 文件：

```bash
systemctl cat nginx
```

查看依赖关系：

```bash
systemctl list-dependencies nginx
```

查看服务属性：

```bash
systemctl show nginx
```

例如：

```bash
systemctl show nginx -p MainPID
```

输出：

```text
MainPID=1023
```

---

## 七、重新加载 systemd

如果修改了：

```
/etc/systemd/system/*.service
```

必须执行：

```bash
systemctl daemon-reload
```

否则修改不会生效。

---

## 八、系统电源管理

关机：

```bash
systemctl poweroff
```

重启：

```bash
systemctl reboot
```

挂起：

```bash
systemctl suspend
```

休眠：

```bash
systemctl hibernate
```

---

## 九、查看启动时间

查看系统启动耗时：

```bash
systemd-analyze
```

例如：

```text
Startup finished in 1.5s (kernel) + 3.8s (userspace)
```

查看耗时最长的服务：

```bash
systemd-analyze blame
```

例如：

```text
2.531s NetworkManager.service
1.882s firewalld.service
```

---

## 十、与 journalctl 配合使用

查看某个服务日志：

```bash
journalctl -u nginx
```

实时查看日志：

```bash
journalctl -u nginx -f
```

查看本次启动后的日志：

```bash
journalctl -u nginx -b
```

---

## 十一、服务文件位置

一般服务定义文件位于：

系统自带：

```text
/usr/lib/systemd/system/
```

或：

```text
/lib/systemd/system/
```

管理员自定义：

```text
/etc/systemd/system/
```

例如：

```
/etc/systemd/system/myapp.service
```

一个典型的 `.service` 文件如下：

```ini
[Unit]
Description=My Application
After=network.target

[Service]
ExecStart=/usr/local/bin/myapp
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

修改后需要执行：

```bash
systemctl daemon-reload
systemctl restart myapp
```

---

## 十二、常见命令速查表

| 操作              | 命令                                         |
| --------------- | ------------------------------------------ |
| 查看状态            | `systemctl status 服务名`                     |
| 启动服务            | `systemctl start 服务名`                      |
| 停止服务            | `systemctl stop 服务名`                       |
| 重启服务            | `systemctl restart 服务名`                    |
| 重新加载配置          | `systemctl reload 服务名`                     |
| 查看是否运行          | `systemctl is-active 服务名`                  |
| 设置开机启动          | `systemctl enable 服务名`                     |
| 取消开机启动          | `systemctl disable 服务名`                    |
| 启动并设为开机启动       | `systemctl enable --now 服务名`               |
| 停止并取消开机启动       | `systemctl disable --now 服务名`              |
| 查看运行中的服务        | `systemctl list-units --type=service`      |
| 查看所有服务          | `systemctl list-unit-files --type=service` |
| 查看失败服务          | `systemctl --failed`                       |
| 查看服务日志          | `journalctl -u 服务名`                        |
| 查看服务配置          | `systemctl cat 服务名`                        |
| 重新加载 systemd 配置 | `systemctl daemon-reload`                  |
| 重启系统            | `systemctl reboot`                         |
| 关机              | `systemctl poweroff`                       |

## 十三、与传统命令对照

| 传统命令（SysV）              | `systemctl` 等效命令          |
| ----------------------- | ------------------------- |
| `service nginx start`   | `systemctl start nginx`   |
| `service nginx stop`    | `systemctl stop nginx`    |
| `service nginx restart` | `systemctl restart nginx` |
| `service nginx status`  | `systemctl status nginx`  |
| `chkconfig nginx on`    | `systemctl enable nginx`  |
| `chkconfig nginx off`   | `systemctl disable nginx` |

目前，除少数较旧的发行版外，`systemctl` 已成为 Linux 服务管理的标准工具。熟练掌握“**status（查看）→ start/stop/restart（控制）→ enable/disable（开机管理）→ journalctl（日志排查）→ daemon-reload（配置生效）**”这一套命令，基本可以满足日常服务器运维和应用部署的需要。