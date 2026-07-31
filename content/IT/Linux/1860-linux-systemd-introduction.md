---
title: Linux Systemd 基础
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKj4
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/186'
---

<img src="images/Linux.svg" width="300">

# Linux Systemd 基础

## 什么是 Systemd

**Systemd** 是 Linux 中最常用的**系统初始化（Init）和服务管理器**。它负责在系统启动时初始化系统环境，并管理系统中的各种后台服务（Service）。

可以把它理解为 Linux 的**总调度员**：

* 系统开机时，它负责启动各种服务。
* 系统运行过程中，它负责监控、停止、重启服务。
* 系统关机时，它负责按照依赖关系有序关闭服务。

目前，绝大多数主流 Linux 发行版（如 Ubuntu、Debian、CentOS 7+、Rocky Linux、AlmaLinux、Fedora、openSUSE 等）都默认使用 Systemd。

---

## 一、Systemd 的基本组成

Systemd 不只是一个程序，而是一整套系统管理框架，主要包括：

| 组件          | 作用         |
| ----------- | ---------- |
| systemd     | 系统初始化和管理核心 |
| systemctl   | 管理服务和系统状态  |
| journalctl  | 查看系统日志     |
| loginctl    | 管理用户登录会话   |
| hostnamectl | 修改主机名      |
| timedatectl | 管理时间和时区    |
| localectl   | 管理语言和键盘布局  |

其中最常用的是：

* **systemctl**
* **journalctl**

---

## 二、Systemd 的工作流程

Linux 开机的大致流程如下：

```text
BIOS/UEFI
      │
Boot Loader（GRUB）
      │
Linux Kernel
      │
systemd（PID 1）
      │
初始化系统
      │
启动各种 Service
      │
进入登录界面
```

可以查看：

```bash
ps -p 1
```

输出类似：

```text
PID TTY      TIME CMD
1 ?          00:01:25 systemd
```

说明 PID=1 的进程就是 systemd。

---

# 三、Systemd 的核心概念——Unit（单元）

Systemd 管理的所有对象都叫 **Unit（单元）**。

可以理解为：

> 一个 Unit 就是一项可以被 Systemd 管理的资源。

例如：

* 一个服务
* 一个挂载点
* 一个定时任务
* 一个设备
* 一个目标（Target）

查看所有 Unit：

```bash
systemctl list-units
```

查看所有 Unit（包括未启动）

```bash
systemctl list-unit-files
```

---

## 四、常见 Unit 类型

Systemd 支持很多 Unit 类型。

| 类型        | 后缀         | 说明        |
| --------- | ---------- | --------- |
| Service   | .service   | 服务（最常见）   |
| Socket    | .socket    | Socket 激活 |
| Target    | .target    | 启动目标      |
| Mount     | .mount     | 挂载点       |
| Device    | .device    | 设备        |
| Timer     | .timer     | 定时器       |
| Path      | .path      | 文件变化触发    |
| Automount | .automount | 自动挂载      |
| Swap      | .swap      | Swap 分区   |

例如：

```text
sshd.service

network.target

cron.service

tmp.mount

logrotate.timer
```

---

# 五、Service（服务）

Service 是最重要的 Unit。

例如：

```text
nginx.service

mysql.service

docker.service

sshd.service
```

查看某个服务：

```bash
systemctl status nginx
```

输出：

```text
● nginx.service
Loaded: loaded
Active: active (running)
Main PID: 1201
```

重点关注：

Loaded

是否安装

Active

运行状态

Main PID

主进程

---

# 六、systemctl 常用命令

## 查看状态

查看服务：

```bash
systemctl status nginx
```

查看所有运行中的服务：

```bash
systemctl list-units --type=service
```

查看失败服务：

```bash
systemctl --failed
```

---

## 启动服务

```bash
systemctl start nginx
```

停止：

```bash
systemctl stop nginx
```

重启：

```bash
systemctl restart nginx
```

重新加载配置：

```bash
systemctl reload nginx
```

重新加载失败再重启：

```bash
systemctl reload-or-restart nginx
```

---

## 开机启动

开启：

```bash
systemctl enable nginx
```

关闭：

```bash
systemctl disable nginx
```

查看：

```bash
systemctl is-enabled nginx
```

---

## 查看是否运行

```bash
systemctl is-active nginx
```

返回：

```text
active
```

或者

```text
inactive
```

---

## 查看依赖

```bash
systemctl list-dependencies nginx
```

---

# 七、Target（启动目标）

以前 Linux 使用 Runlevel：

```text
0 关机

1 单用户

3 命令行

5 图形界面

6 重启
```

Systemd 改成 Target。

对应关系：

| Runlevel | Target            |
| -------- | ----------------- |
| 0        | poweroff.target   |
| 1        | rescue.target     |
| 3        | multi-user.target |
| 5        | graphical.target  |
| 6        | reboot.target     |

查看当前：

```bash
systemctl get-default
```

查看当前运行：

```bash
systemctl list-units --type=target
```

修改默认：

```bash
systemctl set-default multi-user.target
```

恢复图形：

```bash
systemctl set-default graphical.target
```

---

# 八、Journal（日志）

Systemd 自带日志系统。

查看全部日志：

```bash
journalctl
```

查看本次启动：

```bash
journalctl -b
```

上一轮启动：

```bash
journalctl -b -1
```

查看某服务：

```bash
journalctl -u nginx
```

实时查看：

```bash
journalctl -f
```

最近一小时：

```bash
journalctl --since "1 hour ago"
```

今天：

```bash
journalctl --since today
```

错误日志：

```bash
journalctl -p err
```

---

# 九、Service 文件

每个 Service 都有配置文件。

例如：

```text
/usr/lib/systemd/system/nginx.service
```

或者：

```text
/etc/systemd/system/
```

查看：

```bash
systemctl cat nginx
```

一个简单示例：

```ini
[Unit]
Description=My Web Server
After=network.target

[Service]
ExecStart=/usr/local/bin/start.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

主要分为三个部分：

### [Unit]

描述服务。

例如：

```ini
Description=
Documentation=
After=
Requires=
```

---

### [Service]

真正启动程序。

例如：

```ini
ExecStart=
ExecStop=
Restart=
Type=
User=
Group=
WorkingDirectory=
Environment=
```

---

### [Install]

开机启动配置。

例如：

```ini
WantedBy=multi-user.target
```

---

# 十、修改 Service 后需要做什么

修改 service 文件：

```bash
vim my.service
```

重新加载：

```bash
systemctl daemon-reload
```

重启：

```bash
systemctl restart my.service
```

---

# 十一、创建自己的 Service

例如：

```bash
sudo vim /etc/systemd/system/myapp.service
```

写入：

```ini
[Unit]
Description=My App
After=network.target

[Service]
Type=simple
ExecStart=/opt/myapp/run.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

然后：

```bash
systemctl daemon-reload
systemctl enable myapp
systemctl start myapp
```

查看：

```bash
systemctl status myapp
```

---

# 十二、常见故障排查

当服务启动失败时，建议按以下顺序排查：

1. 查看服务状态：

   ```bash
   systemctl status <服务名>
   ```

2. 查看详细日志：

   ```bash
   journalctl -u <服务名> -xe
   ```

3. 检查服务配置：

   ```bash
   systemctl cat <服务名>
   ```

4. 确认启动命令可独立运行：

   ```bash
   <ExecStart 中的命令>
   ```

5. 修改配置后重新加载：

   ```bash
   systemctl daemon-reload
   ```

---

# 十三、学习路线建议

如果希望系统掌握 Systemd，可以按以下顺序学习：

1. **基础命令**

   * `systemctl status`
   * `start`
   * `stop`
   * `restart`
   * `enable`
   * `disable`

2. **Unit 管理**

   * Unit 类型
   * Unit 文件位置
   * Unit 依赖关系

3. **日志分析**

   * `journalctl`
   * 按服务查看日志
   * 按启动次数查看日志

4. **编写 Service**

   * `[Unit]`、`[Service]`、`[Install]`
   * `ExecStart`
   * `Restart`
   * `Type`
   * `User`

5. **高级功能**

   * Timer（替代 cron）
   * Socket Activation（按需启动）
   * Path Unit（监控文件变化）
   * Resource Control（CPU、内存、IO 限制）
   * cgroups 集成
   * `systemd-analyze`（分析启动性能）

## 一个记忆框架

可以用下面这个框架快速理解 Systemd：

```text
                 systemd（PID 1）
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   管理 Unit      管理日志       管理启动流程
        │              │              │
  service、timer   journalctl     target
        │
   systemctl 操作
        │
 start / stop / restart / enable
```

掌握了 **Unit（管理对象）**、**systemctl（管理命令）** 和 **journalctl（日志工具）** 这三个核心概念，就已经具备了日常运维和应用部署所需的大部分 Systemd 基础知识。在此基础上，再学习自定义 Service、Timer 和资源控制等高级功能，会更加得心应手。
