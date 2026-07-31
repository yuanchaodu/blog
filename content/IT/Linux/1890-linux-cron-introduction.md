---
title: Linux Cron 任务调度
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKrQ
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/189'
---

<img src="images/Linux.svg" width="300">

# Linux Cron 任务调度

Cron 是 Linux 中常用的定时任务工具。它像一个“自动闹钟”，到达指定时间后，自动执行命令或脚本，例如数据库备份、日志清理、数据同步和系统巡检。

Cron 服务会读取用户或系统配置的任务表，并按分钟检查是否存在需要执行的任务。任务默认以配置该任务的用户身份运行。([man7.org][1])

## 一、基本组成

Cron 任务通常由两部分组成：

```text
执行时间    执行命令
```

标准用户级 Cron 表达式包含五个时间字段：

```text
分  时  日  月  星期  命令
*   *   *   *   *    command
```

字段含义如下：

| 字段 | 取值范围 | 说明          |
| -- | ---: | ----------- |
| 分钟 | 0～59 | 第几分钟执行      |
| 小时 | 0～23 | 第几个小时执行     |
| 日期 | 1～31 | 每月第几天       |
| 月份 | 1～12 | 第几个月        |
| 星期 |  0～7 | 0和7通常都表示星期日 |
| 命令 |    — | 需要执行的命令或脚本  |

每条有效任务一般由五个时间字段和一条命令组成。系统级配置文件还会多一个“运行用户”字段。([man7.org][1])

例如：

```cron
30 2 * * * /opt/scripts/backup.sh
```

表示每天凌晨 2:30 执行备份脚本。

---

## 二、常用特殊符号

### `*`：任意值

```cron
* * * * * /opt/scripts/test.sh
```

表示每分钟执行一次。

### `,`：多个指定值

```cron
0 8,12,18 * * * /opt/scripts/sync.sh
```

表示每天 8:00、12:00 和 18:00 执行。

### `-`：连续范围

```cron
0 9-18 * * * /opt/scripts/check.sh
```

表示每天 9:00 至 18:00，每小时执行一次。

### `/`：执行间隔

```cron
*/10 * * * * /opt/scripts/check.sh
```

表示每隔 10 分钟执行一次。

也可以组合使用：

```cron
0 */2 * * * /opt/scripts/collect.sh
```

表示从 0 点开始，每隔两小时执行一次。

---

## 三、常用操作命令

### 编辑当前用户的任务

```bash
crontab -e
```

第一次执行时，系统可能要求选择文本编辑器。

### 查看当前用户的任务

```bash
crontab -l
```

### 删除当前用户的全部任务

```bash
crontab -r
```

使用该命令时要谨慎，因为它会直接删除全部用户级 Cron 任务。

### 管理其他用户的任务

```bash
sudo crontab -u username -e
```

### 编辑 root 用户的任务

```bash
sudo crontab -e
```

`crontab` 命令用于安装、查看和删除用户的 Cron 任务表，不建议直接修改 `/var/spool/cron` 下的内部文件。([Debian Manpages][2])

---

## 四、常用调度示例

### 每天凌晨 2 点执行

```cron
0 2 * * * /opt/scripts/backup.sh
```

### 每小时执行一次

```cron
0 * * * * /opt/scripts/check.sh
```

### 每 5 分钟执行一次

```cron
*/5 * * * * /opt/scripts/monitor.sh
```

### 每周一上午 8 点执行

```cron
0 8 * * 1 /opt/scripts/report.sh
```

### 每月 1 日凌晨执行

```cron
0 0 1 * * /opt/scripts/monthly.sh
```

### 工作日每天 18:30 执行

```cron
30 18 * * 1-5 /opt/scripts/daily_report.sh
```

### 每年 1 月 1 日执行

```cron
0 0 1 1 * /opt/scripts/yearly.sh
```

### 每天执行两次

```cron
0 8,20 * * * /opt/scripts/sync.sh
```

### 每隔 30 分钟执行

```cron
*/30 * * * * /opt/scripts/collect.sh
```

---

## 五、特殊时间写法

部分 Cron 实现支持以下简写：

```cron
@reboot   /opt/scripts/startup.sh
@hourly   /opt/scripts/hourly.sh
@daily    /opt/scripts/daily.sh
@weekly   /opt/scripts/weekly.sh
@monthly  /opt/scripts/monthly.sh
@yearly   /opt/scripts/yearly.sh
```

其中：

* `@reboot`：系统启动后执行。
* `@hourly`：每小时执行一次。
* `@daily`：每天执行一次。
* `@weekly`：每周执行一次。
* `@monthly`：每月执行一次。
* `@yearly`：每年执行一次。

不同 Linux 发行版和 Cron 实现可能存在差异，正式使用前应通过本机命令确认：

```bash
man 5 crontab
```

---

## 六、用户级任务与系统级任务

### 用户级任务

通过下面的命令维护：

```bash
crontab -e
```

格式为：

```cron
分 时 日 月 星期 命令
```

例如：

```cron
0 2 * * * /home/user/backup.sh
```

任务以当前用户身份执行。

### 系统级任务

常见配置位置包括：

```text
/etc/crontab
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

系统级 `/etc/crontab` 和 `/etc/cron.d/` 文件需要指定运行用户：

```cron
分 时 日 月 星期 用户 命令
```

例如：

```cron
0 2 * * * root /opt/scripts/backup.sh
```

Cron 通常会读取用户任务表、`/etc/crontab` 和 `/etc/cron.d/` 等位置；Debian 系发行版还常通过系统配置调用 hourly、daily、weekly 和 monthly 目录中的任务。([man7.org][3])

---

## 七、推荐的脚本写法

假设创建备份脚本：

```bash
sudo vi /opt/scripts/backup.sh
```

脚本内容：

```bash
#!/bin/bash

set -u

BACKUP_DIR="/data/backup"
SOURCE_DIR="/data/app"
LOG_FILE="/var/log/app-backup.log"
DATE="$(date '+%Y%m%d_%H%M%S')"

mkdir -p "$BACKUP_DIR"

if tar -czf "$BACKUP_DIR/app_$DATE.tar.gz" "$SOURCE_DIR" >>"$LOG_FILE" 2>&1; then
    echo "$(date '+%F %T') backup completed" >>"$LOG_FILE"
else
    echo "$(date '+%F %T') backup failed" >>"$LOG_FILE"
    exit 1
fi
```

增加执行权限：

```bash
sudo chmod +x /opt/scripts/backup.sh
```

先手工测试：

```bash
sudo /opt/scripts/backup.sh
```

确认正常后，再加入 Cron：

```cron
0 2 * * * /opt/scripts/backup.sh
```

这个顺序很重要：**先手工运行成功，再设置定时执行。**

---

## 八、日志输出处理

Cron 执行命令时，程序可能产生标准输出和错误输出。部分 Cron 实现会尝试通过本地邮件发送输出，但很多服务器没有配置邮件服务，因此建议主动写入日志。([man7.org][3])

### 标准输出和错误输出写入同一文件

```cron
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

其中：

```text
>>    追加标准输出
2>&1  将错误输出也写入同一位置
```

### 不保留任何输出

```cron
0 2 * * * /opt/scripts/backup.sh >/dev/null 2>&1
```

重要任务不建议完全丢弃输出，否则失败时难以排查。

### 每天生成独立日志

Cron 中 `%` 具有特殊含义，直接使用 `date +%F` 可能产生意外结果。可以转义 `%`：

```cron
0 2 * * * /opt/scripts/backup.sh >> "/var/log/backup-$(date +\%F).log" 2>&1
```

更稳妥的方法，是把日期和日志处理放进脚本内部。

---

## 九、Cron 中最常见的问题

### 1. 环境变量不同

在命令行可以执行，不代表在 Cron 中也能执行。Cron 的运行环境通常比较简单，`PATH` 可能不完整。

不推荐：

```cron
0 2 * * * python backup.py
```

推荐使用绝对路径：

```cron
0 2 * * * /usr/bin/python3 /opt/scripts/backup.py
```

也可以在 crontab 开头设置环境变量：

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * * /opt/scripts/backup.sh
```

### 2. 工作目录不同

Cron 不一定从脚本所在目录执行。

例如，下面的写法容易出错：

```bash
python3 app.py
```

建议：

```cron
0 2 * * * cd /opt/app && /usr/bin/python3 app.py
```

或者在脚本中明确切换目录：

```bash
cd /opt/app || exit 1
```

### 3. 脚本没有执行权限

检查权限：

```bash
ls -l /opt/scripts/backup.sh
```

增加权限：

```bash
chmod +x /opt/scripts/backup.sh
```

### 4. 脚本首行解释器错误

Shell 脚本建议使用：

```bash
#!/bin/bash
```

或：

```bash
#!/usr/bin/env bash
```

同时注意脚本不能包含 Windows 风格换行符。可使用下面的命令检查：

```bash
file /opt/scripts/backup.sh
```

### 5. 运行用户权限不足

普通用户的 Cron 任务无法自动获得 root 权限。需要系统权限的任务，应放入 root 的 crontab，或者使用 systemd 服务进行管理。

### 6. 任务重复运行

如果上一次任务尚未完成，下一次又被启动，可能产生重复处理或数据冲突。

可使用 `flock` 加锁：

```cron
*/5 * * * * /usr/bin/flock -n /var/run/data-sync.lock /opt/scripts/sync.sh >> /var/log/data-sync.log 2>&1
```

`-n` 表示锁已被占用时立即退出，不等待。

### 7. 日期与星期同时设置

Cron 对“每月日期”和“星期”字段的处理容易被误解。某些实现中，当两个字段都不是 `*` 时，只要其中一个匹配，任务就可能执行。编写复杂规则前，应以本机 `man 5 crontab` 为准。([man7.org][1])

---

## 十、服务状态检查

不同发行版的服务名称可能不同。

### Debian、Ubuntu

```bash
systemctl status cron
sudo systemctl enable --now cron
```

### RHEL、Rocky Linux、AlmaLinux、CentOS

```bash
systemctl status crond
sudo systemctl enable --now crond
```

查看进程：

```bash
ps -ef | grep -E '[c]ron|[c]rond'
```

---

## 十一、排错方法

### 第一步：确认任务已保存

```bash
crontab -l
```

### 第二步：手工执行完整命令

```bash
/bin/bash /opt/scripts/backup.sh
```

不要只测试脚本的一小部分，要测试 Cron 中写入的完整命令。

### 第三步：临时改成每分钟执行

```cron
* * * * * /opt/scripts/backup.sh >> /tmp/backup-test.log 2>&1
```

测试完成后，应及时恢复正式时间，避免任务持续执行。

### 第四步：查看日志

Debian、Ubuntu 常见方式：

```bash
journalctl -u cron
```

或者：

```bash
grep CRON /var/log/syslog
```

RHEL 系常见方式：

```bash
journalctl -u crond
```

或者：

```bash
grep CROND /var/log/cron
```

### 第五步：在任务中记录环境

```cron
* * * * * /usr/bin/env > /tmp/cron-env.txt
```

然后将其与命令行环境比较：

```bash
env
```

这对排查 `PATH`、语言环境和用户变量问题很有效。

---

## 十二、生产环境建议

生产环境中的 Cron 任务建议遵循以下原则：

1. 命令和脚本使用绝对路径。
2. 脚本先手工测试，再配置定时任务。
3. 重要任务记录开始时间、结束时间和执行结果。
4. 使用 `flock` 防止任务重复运行。
5. 设置超时，避免脚本永久卡住。
6. 备份、删除等任务要检查返回码。
7. 不要把数据库密码直接写在 crontab 中。
8. 定期清理日志，避免磁盘空间耗尽。
9. 对失败任务设置监控或告警。
10. 对关键任务优先考虑 systemd timer。

带超时和锁的示例：

```cron
*/10 * * * * /usr/bin/flock -n /run/data-sync.lock /usr/bin/timeout 20m /opt/scripts/data-sync.sh >> /var/log/data-sync.log 2>&1
```

---

## 十三、Cron 与 systemd Timer 的区别

现代 Linux 系统还可以使用 systemd Timer。`.timer` 单元负责确定执行时间，通常触发对应的 `.service` 单元。([自由桌面][4])

| 对比项     | Cron    | systemd Timer         |
| ------- | ------- | --------------------- |
| 配置难度    | 简单      | 配置文件较多                |
| 适合任务    | 简单周期任务  | 重要系统任务                |
| 日志      | 需要自行重定向 | 自动进入 journal          |
| 依赖控制    | 较弱      | 可依赖网络、挂载点和其他服务        |
| 失败状态    | 不易集中查看  | 可通过 systemctl 查看      |
| 错过任务补执行 | 通常不处理   | 可使用 `Persistent=true` |
| 随机延迟    | 一般需自行实现 | 原生支持                  |
| 权限隔离    | 主要依赖用户  | 可使用服务安全配置             |

例如，服务器每天凌晨 2 点关机维护。如果 Cron 任务安排在 2:30，它通常会被错过。systemd Timer 可以通过 `Persistent=true`，在服务器恢复后补执行错过的任务。systemd Timer 同时支持日历时间和相对于启动时间的调度。([自由桌面][4])

因此：

* 简单、低风险的定时脚本，可以使用 Cron。
* 涉及服务依赖、失败追踪、补执行和安全隔离的任务，更适合 systemd Timer。

## 总结

Cron 的核心格式是：

```cron
分 时 日 月 星期 命令
```

一个较规范的生产任务可以写成：

```cron
0 2 * * * /usr/bin/flock -n /run/backup.lock /usr/bin/timeout 2h /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

它实现了：

* 每天凌晨 2 点执行；
* 防止重复运行；
* 最长运行两小时；
* 保存执行日志。

掌握 Cron 的重点不只是“把时间写对”，还要注意运行用户、环境变量、绝对路径、日志、超时和重复执行问题。

[1]: https://man7.org/linux/man-pages/man5/crontab.5.html "crontab(5) - Linux manual page"
[2]: https://manpages.debian.org/experimental/cronie/crontab.1.en.html "crontab(1) — cronie — Debian experimental"
[3]: https://man7.org/linux/man-pages/man8/cron.8.html "cron(8) - Linux manual page"
[4]: https://www.freedesktop.org/software/systemd/man/systemd.timer.html "systemd.timer"
