---
title: Linux 时间管理
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKu0
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/192'
---

<img src="images/Linux.svg" width="300">

# Linux 时间管理

Linux 时间管理主要涉及四个部分：**系统时间、硬件时间、时区和网络时间同步**。服务器时间一旦不准确，可能导致日志顺序混乱、证书认证失败、数据库数据异常以及集群节点通信故障。

## 一、Linux 中的几种时间

### 1. 系统时间

系统时间由 Linux 内核维护，系统启动后持续运行。应用程序、日志、数据库和文件时间戳通常都使用系统时间。

查看系统时间：

```bash
date
```

按照指定格式显示：

```bash
date "+%Y-%m-%d %H:%M:%S"
```

输出示例：

```text
2026-07-31 17:00:00
```

`date` 命令也可以解析和计算日期，例如：

```bash
date -d "tomorrow"
date -d "7 days ago"
date -d "2026-08-01 +1 month"
```

`date -s` 可以直接修改系统时间，但生产服务器一般不建议在时间同步服务运行时手工修改。([man7.org][1])

---

### 2. 硬件时间

硬件时间也称为 **RTC 时间**或 **BIOS 时间**。它由主板上的实时时钟芯片维护，即使服务器关机，通常也会依靠电池继续计时。系统启动时，会参考硬件时间初始化系统时间。([Linux内核文档][2])

查看硬件时间：

```bash
sudo hwclock --show
```

将系统时间写入硬件时钟：

```bash
sudo hwclock --systohc
```

简写：

```bash
sudo hwclock -w
```

将硬件时间写入系统时间：

```bash
sudo hwclock --hctosys
```

简写：

```bash
sudo hwclock -s
```

`hwclock` 可以查看和设置硬件时间，也可以在系统时间与硬件时间之间进行同步。([man7.org][3])

---

### 3. 时区

时区决定系统如何把统一时间转换为当地时间。

查看当前时间及时区：

```bash
timedatectl
```

典型输出：

```text
Local time: Fri 2026-07-31 17:00:00 CST
Universal time: Fri 2026-07-31 09:00:00 UTC
RTC time: Fri 2026-07-31 09:00:00
Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
```

查看支持的时区：

```bash
timedatectl list-timezones
```

搜索中国相关时区：

```bash
timedatectl list-timezones | grep Shanghai
```

设置中国标准时区：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```

检查时区文件：

```bash
ls -l /etc/localtime
cat /etc/timezone
```

其中，`/etc/timezone` 并非所有发行版都使用，`timedatectl` 通常是 systemd 系统中更统一的管理方式。`timedatectl` 可以查询和修改系统时间、时区以及网络时间同步状态。([自由桌面][4])

## 二、UTC 与本地时间

### UTC

UTC 是统一的时间基准，不随国家和地区变化。

### 本地时间

本地时间是在 UTC 基础上加上时区偏移。例如：

```text
北京时间 = UTC + 8小时
```

通常建议：

* Linux 服务器的硬件时钟使用 UTC。
* 操作系统设置正确时区。
* 应用程序内部尽量保存 UTC 时间。
* 展示数据时再转换为当地时间。

查看硬件时钟是否使用本地时间：

```bash
timedatectl
```

重点查看：

```text
RTC in local TZ: no
```

`no` 表示硬件时钟通常按 UTC 保存，这是 Linux 服务器更常见的设置。

将硬件时钟设置为 UTC 模式：

```bash
sudo timedatectl set-local-rtc 0
```

不建议在没有特殊需求时设置为本地时间模式：

```bash
sudo timedatectl set-local-rtc 1
```

## 三、手工设置时间

### 使用 timedatectl

关闭自动同步：

```bash
sudo timedatectl set-ntp false
```

设置时间：

```bash
sudo timedatectl set-time "2026-07-31 17:10:00"
```

将系统时间写入硬件时钟：

```bash
sudo hwclock --systohc
```

重新开启自动同步：

```bash
sudo timedatectl set-ntp true
```

### 使用 date

```bash
sudo date -s "2026-07-31 17:10:00"
```

设置完成后，可以写入硬件时钟：

```bash
sudo hwclock -w
```

手工修改时间可能让系统时间突然向前或向后跳变。数据库、日志系统、认证系统和集群环境中应谨慎操作。

## 四、网络时间同步

生产环境中，通常通过 NTP 协议从时间服务器自动校准时间。

Linux 常见的时间同步组件有：

* `systemd-timesyncd`
* `chrony`
* `ntpd`

一般只保留一套时间同步服务，避免多个服务同时调整系统时间。

### 1. systemd-timesyncd

`systemd-timesyncd` 是 systemd 提供的网络时间同步服务，可以使用远程 NTP 服务器同步本机系统时间。([自由桌面][5])

启用自动同步：

```bash
sudo timedatectl set-ntp true
```

查看状态：

```bash
timedatectl status
timedatectl timesync-status
```

查看服务：

```bash
systemctl status systemd-timesyncd
```

查看日志：

```bash
journalctl -u systemd-timesyncd
```

主要配置文件：

```text
/etc/systemd/timesyncd.conf
```

配置示例：

```ini
[Time]
NTP=ntp1.example.com ntp2.example.com
FallbackNTP=ntp.example.net
```

修改后重新启动：

```bash
sudo systemctl restart systemd-timesyncd
```

`timesyncd.conf` 用于配置 NTP 服务器和网络时间同步参数。([自由桌面][6])

---

### 2. chrony

在服务器、虚拟机以及网络质量不稳定的环境中，通常会使用 Chrony。

查看服务状态：

```bash
systemctl status chronyd
```

部分 Debian、Ubuntu 系统的服务名称可能显示为：

```bash
systemctl status chrony
```

查看同步状态：

```bash
chronyc tracking
```

查看时间源：

```bash
chronyc sources -v
```

查看源统计信息：

```bash
chronyc sourcestats -v
```

立即加快校准：

```bash
sudo chronyc makestep
```

常见配置文件：

```text
/etc/chrony.conf
```

或者：

```text
/etc/chrony/chrony.conf
```

企业内网配置示例：

```ini
server 192.168.10.10 iburst
server 192.168.10.11 iburst

makestep 1.0 3
rtcsync
```

其中：

* `server`：指定上级时间服务器。
* `iburst`：启动时快速发送一组请求，加快首次同步。
* `makestep`：在符合条件时允许直接修正较大的时间偏差。
* `rtcsync`：让内核定期把同步后的系统时间更新到硬件时钟。

## 五、常用检查命令

```bash
# 查看当前日期和时间
date

# 查看 UTC 时间
date -u

# 查看系统时间、时区和同步状态
timedatectl

# 查看硬件时间
sudo hwclock --show

# 查看 chrony 同步状态
chronyc tracking

# 查看 chrony 时间源
chronyc sources -v

# 查看 timesyncd 状态
timedatectl timesync-status

# 查看时间同步相关服务
systemctl status chronyd
systemctl status chrony
systemctl status systemd-timesyncd

# 查看当前时区文件
ls -l /etc/localtime

# 查看系统启动时间
uptime -s

# 查看系统运行时长
uptime
```

## 六、时间问题排查

### 情况一：时间相差整整几个小时

这通常是时区错误，或者硬件时钟的 UTC、本地时间模式设置不一致。

检查：

```bash
timedatectl
date
date -u
sudo hwclock --show
```

修正时区：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```

设置 RTC 使用 UTC：

```bash
sudo timedatectl set-local-rtc 0
sudo hwclock --systohc --utc
```

---

### 情况二：时间每天慢几秒或快几秒

通常是没有启用网络时间同步，或者 NTP 服务器不可访问。

检查：

```bash
timedatectl
chronyc tracking
chronyc sources -v
```

检查网络和域名解析：

```bash
ping <NTP服务器地址>
getent hosts <NTP服务器域名>
```

还应检查防火墙是否允许 NTP 通信。传统 NTP 通常使用 UDP 123 端口。

---

### 情况三：显示 NTP 已启用，但没有完成同步

检查：

```bash
timedatectl
timedatectl timesync-status
journalctl -u systemd-timesyncd --since today
```

Chrony 环境检查：

```bash
chronyc tracking
chronyc sources -v
journalctl -u chronyd --since today
```

常见原因包括：

* NTP 服务器地址错误。
* DNS 解析失败。
* 防火墙阻断。
* 上级时间服务器不可用。
* 本机时间偏差过大。
* 同时运行了多个时间同步服务。

---

### 情况四：重启后时间又不正确

重点检查硬件时钟：

```bash
sudo hwclock --show
timedatectl
```

将正确的系统时间写入硬件时钟：

```bash
sudo hwclock --systohc
```

如果每次断电后硬件时间都恢复异常，可能是主板 RTC 电池、虚拟化平台时间策略或宿主机时间存在问题。

## 七、虚拟机和容器注意事项

### 虚拟机

虚拟机时间可能同时受到以下因素影响：

* 虚拟机内部的 NTP 服务。
* 宿主机时间。
* 虚拟化平台的时间同步功能。
* 虚拟 CPU 调度延迟。
* 虚拟化时钟源。

Linux 内核为 KVM、Hyper-V 等虚拟化平台提供相应的虚拟时钟机制。([Linux内核文档][7])

生产环境中，应明确由哪一层负责同步时间，避免宿主机同步机制与虚拟机内部 NTP 同时频繁调整时间。

### 容器

容器通常共享宿主机内核时间，因此一般不能独立维护完整的系统时钟。容器时间异常时，应优先检查：

```bash
# 宿主机
date
timedatectl

# 容器
date
```

容器与宿主机相差整整几个小时，很多时候是容器缺少正确的时区文件，而不是系统时间本身错误。

## 八、企业环境建议

企业内部可以采用分层时间同步架构：

```text
外部权威时间源
       ↓
企业内部主时间服务器
       ↓
备用时间服务器
       ↓
应用服务器、数据库、网络设备和终端
```

建议至少做到：

1. 配置两个或更多内部时间源。
2. 所有服务器统一使用同一套时间标准。
3. 硬件时钟统一使用 UTC。
4. 业务系统内部尽量保存 UTC 时间。
5. 监控服务器时间偏差和同步状态。
6. 禁止普通用户随意修改系统时间。
7. 数据库、域控、证书系统和日志平台优先纳入监控。
8. 不要同时启用 Chrony、ntpd 和 systemd-timesyncd。

可以定期采集以下信息：

```bash
timedatectl
chronyc tracking
chronyc sources -v
```

重点关注：

* 是否已完成同步。
* 当前时间源是否可达。
* 时间偏差是否持续增大。
* 是否发生时间源切换。
* 是否存在系统时间突然跳变。

## 九、快速操作示例

在采用 systemd 的普通 Linux 服务器上，可按下面步骤设置：

```bash
# 设置时区
sudo timedatectl set-timezone Asia/Shanghai

# 设置硬件时钟采用 UTC
sudo timedatectl set-local-rtc 0

# 开启网络时间同步
sudo timedatectl set-ntp true

# 查看最终状态
timedatectl
```

若服务器使用 Chrony：

```bash
sudo systemctl enable --now chronyd
chronyc tracking
chronyc sources -v
```

可以把 Linux 时间管理理解为三块钟表：

* **RTC 是断电后仍在走的钟。**
* **系统时间是操作系统正在使用的钟。**
* **NTP 是定期帮助系统对表的标准钟。**

只要时区正确、RTC 模式统一、网络同步正常，Linux 的时间管理通常就会比较稳定。

[1]: https://man7.org/linux/man-pages/man1/date.1.html "date(1) - Linux manual page"
[2]: https://docs.kernel.org/admin-guide/rtc.html "Real Time Clock (RTC) Drivers for Linux"
[3]: https://man7.org/linux/man-pages/man8/hwclock.8.html "hwclock(8) - Linux manual page"
[4]: https://www.freedesktop.org/software/systemd/man/timedatectl.html "timedatectl"
[5]: https://www.freedesktop.org/software/systemd/man/systemd-timesyncd.service.html "systemd-timesyncd.service"
[6]: https://www.freedesktop.org/software/systemd/man/timesyncd.conf.html "timesyncd.conf"
[7]: https://docs.kernel.org/timers/timekeeping.html "Clock sources, Clock events, sched_clock() and delay timers"
