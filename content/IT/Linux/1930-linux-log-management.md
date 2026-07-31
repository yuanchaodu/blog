---
title: Linux 日志管理
section: IT
category: Linux
---

<img src="images/Linux.svg" width="300">

# Linux 日志管理

Linux 日志管理主要解决四个问题：

1. 系统发生了什么。
2. 问题发生在什么时间。
3. 是哪个服务或用户引起的。
4. 日志如何保存、清理和集中分析。

现代 Linux 通常同时使用 **systemd-journald、rsyslog 和 logrotate**。它们可以理解为：

* `systemd-journald`：负责收集日志。
* `rsyslog`：负责分类、转发和集中存储日志。
* `logrotate`：负责压缩、轮转和删除旧日志。

## 一、Linux 日志存放位置

传统文本日志通常位于：

```text
/var/log/
```

常见日志包括：

| 日志文件                       | 主要内容                     |
| -------------------------- | ------------------------ |
| `/var/log/messages`        | 系统综合日志，常见于 RHEL、CentOS   |
| `/var/log/syslog`          | 系统综合日志，常见于 Ubuntu、Debian |
| `/var/log/secure`          | 登录、认证、sudo 等安全日志         |
| `/var/log/auth.log`        | Ubuntu、Debian 的认证日志      |
| `/var/log/cron`            | 定时任务日志                   |
| `/var/log/dmesg`           | 内核启动和硬件检测信息              |
| `/var/log/audit/audit.log` | Linux Audit 审计日志         |
| `/var/log/nginx/`          | Nginx 访问和错误日志            |
| `/var/log/httpd/`          | Apache 日志                |

不同发行版的文件名称可能不同，应以实际配置为准。

可以先查看日志目录：

```bash
sudo ls -lh /var/log
```

查看占用空间最大的日志：

```bash
sudo du -ah /var/log | sort -rh | head -20
```

## 二、使用 journalctl 查看日志

现代 Linux 大多使用 `systemd-journald` 收集内核、系统服务、标准输出和传统 syslog 消息。日志采用结构化、带索引的方式保存，因此可以按服务、时间、级别等条件查询。([自由桌面][1])

### 1. 查看全部日志

```bash
sudo journalctl
```

默认从较早的日志开始显示。

查看最新日志：

```bash
sudo journalctl -e
```

实时查看：

```bash
sudo journalctl -f
```

效果类似：

```bash
tail -f /var/log/messages
```

### 2. 查看某个服务

例如查看 SSH 服务：

```bash
sudo journalctl -u sshd
```

实时查看 Nginx：

```bash
sudo journalctl -u nginx -f
```

查看服务本次启动以来的日志：

```bash
sudo journalctl -u nginx -b
```

### 3. 按时间查看

查看今天的日志：

```bash
sudo journalctl --since today
```

查看最近一小时：

```bash
sudo journalctl --since "1 hour ago"
```

查看指定时间段：

```bash
sudo journalctl \
  --since "2026-07-31 08:00:00" \
  --until "2026-07-31 12:00:00"
```

### 4. 按日志级别查看

只查看错误日志：

```bash
sudo journalctl -p err
```

常见日志级别从严重到普通依次为：

```text
emerg
alert
crit
err
warning
notice
info
debug
```

查看警告及以上级别：

```bash
sudo journalctl -p warning
```

### 5. 查看内核和启动日志

查看内核日志：

```bash
sudo journalctl -k
```

查看本次启动：

```bash
sudo journalctl -b
```

查看上一次启动：

```bash
sudo journalctl -b -1
```

列出历史启动记录：

```bash
sudo journalctl --list-boots
```

### 6. 查看磁盘占用

```bash
sudo journalctl --disk-usage
```

清理到只保留最近七天：

```bash
sudo journalctl --vacuum-time=7d
```

限制日志总容量：

```bash
sudo journalctl --vacuum-size=1G
```

## 三、配置 systemd-journald

主配置文件为：

```text
/etc/systemd/journald.conf
```

也可以通过以下目录增加独立配置片段：

```text
/etc/systemd/journald.conf.d/
```

这是当前官方推荐支持的配置方式之一。([man7.org][2])

建议使用配置片段，而不是直接修改主文件：

```bash
sudo mkdir -p /etc/systemd/journald.conf.d
sudo vi /etc/systemd/journald.conf.d/10-storage.conf
```

示例：

```ini
[Journal]
Storage=persistent
SystemMaxUse=2G
SystemKeepFree=5G
MaxRetentionSec=30day
Compress=yes
```

参数含义：

* `Storage=persistent`：日志持久保存，系统重启后仍可查询。
* `SystemMaxUse=2G`：日志最多占用 2GB。
* `SystemKeepFree=5G`：至少为磁盘保留 5GB 空间。
* `MaxRetentionSec=30day`：最长保留 30 天。
* `Compress=yes`：压缩较旧日志。

应用配置：

```bash
sudo systemctl restart systemd-journald
```

检查服务：

```bash
systemctl status systemd-journald
```

持久化日志一般保存在：

```text
/var/log/journal/
```

运行期临时日志通常保存在：

```text
/run/log/journal/
```

## 四、使用 rsyslog 管理文本日志

`rsyslog` 是一个日志收集、过滤、转换和转发工具。它既可以把日志写入本地文件，也可以把多台服务器日志转发到中央日志服务器。([Rsyslog 文档][3])

主要配置位置：

```text
/etc/rsyslog.conf
/etc/rsyslog.d/*.conf
```

([Rsyslog 文档][4])

查看服务状态：

```bash
systemctl status rsyslog
```

重启服务：

```bash
sudo systemctl restart rsyslog
```

检查配置语法：

```bash
sudo rsyslogd -N1
```

### 1. facility 和 severity

传统 syslog 规则常采用：

```text
facility.severity    保存位置
```

例如：

```text
authpriv.*           /var/log/secure
cron.*               /var/log/cron
*.err                /var/log/error.log
```

`facility` 表示日志来源，例如：

```text
auth
authpriv
cron
daemon
kern
mail
user
local0～local7
```

`severity` 表示严重程度，例如：

```text
emerg
alert
crit
err
warning
notice
info
debug
```

### 2. 为应用单独保存日志

创建配置：

```bash
sudo vi /etc/rsyslog.d/myapp.conf
```

内容：

```text
local0.*    /var/log/myapp.log
```

应用程序可以通过 `logger` 命令测试：

```bash
logger -p local0.info "My application test message"
```

查看结果：

```bash
tail -f /var/log/myapp.log
```

## 五、使用 logrotate 轮转日志

日志如果长期不清理，会不断增长并占满磁盘。`logrotate` 可以按时间或文件大小轮转日志，并完成压缩、保留和删除。([GitHub][5])

主要配置位置：

```text
/etc/logrotate.conf
/etc/logrotate.d/
```

例如，为应用日志创建配置：

```bash
sudo vi /etc/logrotate.d/myapp
```

内容：

```text
/var/log/myapp/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    dateext
    create 0640 root adm
}
```

参数含义：

| 参数              | 作用         |
| --------------- | ---------- |
| `daily`         | 每天轮转       |
| `weekly`        | 每周轮转       |
| `size 100M`     | 达到100MB时轮转 |
| `rotate 30`     | 保留30个历史文件  |
| `compress`      | 压缩旧日志      |
| `delaycompress` | 延迟一个周期再压缩  |
| `missingok`     | 文件不存在时不报错  |
| `notifempty`    | 空文件不轮转     |
| `dateext`       | 文件名增加日期    |
| `create`        | 创建新的日志文件   |

检查配置：

```bash
sudo logrotate -d /etc/logrotate.conf
```

`-d` 只模拟，不会真正轮转。

强制执行：

```bash
sudo logrotate -f /etc/logrotate.conf
```

生产环境中，执行强制轮转前应确认配置正确。

## 六、常见日志分析命令

### 实时查看文件

```bash
tail -f /var/log/messages
```

### 查看最后100行

```bash
tail -n 100 /var/log/secure
```

### 搜索错误信息

```bash
grep -i "error" /var/log/messages
```

同时匹配多个关键词：

```bash
grep -Ei "error|failed|critical" /var/log/messages
```

### 统计重复信息

```bash
grep "Failed password" /var/log/secure |
awk '{print $(NF-3)}' |
sort |
uniq -c |
sort -nr |
head
```

这个命令可以粗略统计 SSH 登录失败来源。不同系统的日志格式可能不同，需要根据实际字段调整。

### 查看压缩日志

```bash
zless /var/log/messages-20260730.gz
```

搜索压缩日志：

```bash
zgrep -i "error" /var/log/*.gz
```

### 同时查看 journal 和服务状态

```bash
systemctl status nginx
sudo journalctl -u nginx --since "30 minutes ago"
```

## 七、集中日志管理

当服务器数量较多时，不建议逐台登录查看日志。可以建立集中日志平台：

```text
业务服务器
   ↓
journald / rsyslog / Agent
   ↓
中央日志接收服务器
   ↓
日志存储、检索、看板和告警
```

常见方案包括：

* rsyslog 中央日志服务器
* Elastic Stack
* Grafana Loki
* OpenSearch
* Graylog
* 企业 SIEM 平台

rsyslog 支持过滤、结构化处理、队列缓冲，以及向 Elasticsearch、Kafka 等后端转发，适合承担日志采集和中转任务。([Rsyslog 文档][3])

生产环境建议优先使用：

```text
TCP + TLS
```

而不是只使用 UDP。原因是 UDP 不保证日志一定送达，也没有传输加密。

## 八、企业环境管理建议

### 1. 分类管理

至少分为：

* 操作系统日志
* 安全认证日志
* 数据库日志
* 应用系统日志
* 网络设备日志
* 审计日志
* 工业控制和生产系统日志

### 2. 设置统一时间

所有服务器、网络设备和应用系统应使用统一的 NTP 时间源。否则不同设备的日志时间不一致，故障分析就像把几块走时不同的手表放在一起，很难还原事件顺序。

### 3. 防止磁盘写满

重点监控：

```text
/var/log
/var/log/journal
应用程序日志目录
数据库日志目录
```

建议设置：

* journal 最大容量。
* logrotate 保留周期。
* 文件系统空间告警。
* 单个日志文件大小告警。
* 日志异常增长告警。

### 4. 保护日志完整性

日志文件应限制普通用户修改：

```bash
sudo chown root:adm /var/log/myapp.log
sudo chmod 640 /var/log/myapp.log
```

对于重要审计日志，还应：

* 集中异地保存。
* 加密传输。
* 限制删除权限。
* 记录日志平台自身的操作。
* 按安全和合规要求设置保存期限。

### 5. 不记录敏感信息

应用日志中不应直接记录：

* 用户密码。
* 身份认证令牌。
* 数据库连接密码。
* 完整身份证号。
* 银行卡信息。
* 私钥和密钥。
* 敏感生产配方或工艺参数。

## 九、故障排查基本流程

遇到服务异常时，可以按以下顺序检查：

```bash
# 1. 查看服务状态
systemctl status 服务名

# 2. 查看最近日志
journalctl -u 服务名 -n 100

# 3. 查看指定时间段
journalctl -u 服务名 \
  --since "2026-07-31 15:00:00"

# 4. 查看系统错误
journalctl -p err --since today

# 5. 查看内核问题
journalctl -k --since today

# 6. 检查磁盘空间
df -h
df -i

# 7. 检查日志目录大小
du -sh /var/log/*
```

## 十、推荐的基础配置目标

对于普通企业 Linux 服务器，可以采用以下基线：

```text
日志持久保存：开启
本机日志保留：30～90天
旧日志压缩：开启
日志空间上限：设置
安全日志集中上传：开启
传输加密：开启
服务器时间同步：统一
磁盘空间告警：开启
日志平台权限审计：开启
```

总体来说，Linux 日志管理不只是“查看文件”，而是一套完整流程：

```text
产生日志 → 收集日志 → 分类保存 → 定期轮转
→ 集中传输 → 检索分析 → 异常告警 → 安全审计
```

单台服务器重点掌握 `journalctl` 和 `logrotate`；服务器数量增加后，应进一步建设集中日志平台。

[1]: https://www.freedesktop.org/software/systemd/man/journalctl.html "journalctl"
[2]: https://man7.org/linux/man-pages/man5/journald.conf.5.html "journald.conf(5) - Linux manual page"
[3]: https://docs.rsyslog.com/doc/index.html "rsyslog 8 daily stable documentation"
[4]: https://docs.rsyslog.com/doc/configuration/index.html "Configuration - rsyslog 8 daily stable documentation"
[5]: https://github.com/logrotate/logrotate/blob/main/README.md "logrotate/README.md at main"