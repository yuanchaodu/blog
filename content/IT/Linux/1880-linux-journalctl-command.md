---
title: Linux journalctl 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKnv
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/188'
---

<img src="images/Linux.svg" width="300">

# Linux journalctl 命令

`journalctl` 是 Linux 中 **systemd 日志系统（systemd-journald）** 的日志查看工具，用于查看系统、内核、服务和应用程序的日志。对于使用 **systemd** 的发行版（如 Ubuntu、Debian、CentOS 7+/RHEL 7+、Rocky Linux、AlmaLinux、Fedora 等），它是排查问题最重要的命令之一。([Linuxize][1])

## 基本语法

```bash
journalctl [选项] [过滤条件]
```

---

## 常用命令

### 1. 查看所有日志

```bash
journalctl
```

默认从最早的日志开始显示，使用 `↑`、`↓`、`PageUp`、`PageDown` 浏览，按 **q** 退出。([Linuxize][1])

---

### 2. 查看最新日志

显示最后 100 行：

```bash
journalctl -n 100
```

显示最后 20 行：

```bash
journalctl -n 20
```

---

### 3. 实时查看日志（类似 tail -f）

```bash
journalctl -f
```

如果只查看某个服务：

```bash
journalctl -u nginx -f
```

---

## 按服务查看日志

例如查看 SSH：

```bash
journalctl -u ssh
```

CentOS/RHEL：

```bash
journalctl -u sshd
```

查看 Docker：

```bash
journalctl -u docker
```

查看 Nginx：

```bash
journalctl -u nginx
```

查看 MySQL：

```bash
journalctl -u mysqld
```

---

## 按时间查看

查看今天

```bash
journalctl --since today
```

查看昨天

```bash
journalctl --since yesterday
```

查看最近一小时

```bash
journalctl --since "1 hour ago"
```

指定时间段

```bash
journalctl \
--since "2026-07-31 10:00:00" \
--until "2026-07-31 12:00:00"
```

---

## 查看本次启动日志

```bash
journalctl -b
```

查看上一次启动

```bash
journalctl -b -1
```

查看所有启动记录

```bash
journalctl --list-boots
```

这对于分析服务器异常重启、系统崩溃等问题非常有用。([Linuxize][1])

---

## 查看内核日志

```bash
journalctl -k
```

查看本次启动的内核日志

```bash
journalctl -k -b
```

---

## 按日志级别过滤

| 级别 | 含义           |
| -- | ------------ |
| 0  | emerg（系统不可用） |
| 1  | alert        |
| 2  | crit         |
| 3  | err（错误）      |
| 4  | warning（警告）  |
| 5  | notice       |
| 6  | info（信息）     |
| 7  | debug（调试）    |

例如：

只查看错误：

```bash
journalctl -p err
```

查看警告及以上：

```bash
journalctl -p warning
```

查看调试信息：

```bash
journalctl -p debug
```

---

## 搜索关键字

例如搜索 "failed"：

```bash
journalctl -g failed
```

搜索 "error"：

```bash
journalctl -g error
```

---

## 查看指定进程

例如：

```bash
journalctl _PID=12345
```

---

## 指定输出格式

默认格式：

```bash
journalctl
```

ISO 时间格式：

```bash
journalctl -o short-iso
```

JSON：

```bash
journalctl -o json-pretty
```

只输出日志正文：

```bash
journalctl -o cat
```

---

## 查看日志占用空间

```bash
journalctl --disk-usage
```

例如输出：

```
Archived and active journals take up 1.3G in the file system.
```

---

## 清理日志

保留 500MB：

```bash
sudo journalctl --vacuum-size=500M
```

保留最近 7 天：

```bash
sudo journalctl --vacuum-time=7d
```

---

## 运维中最常用的组合

### 查看服务启动失败原因

```bash
journalctl -u nginx -b
```

---

### 查看最近 100 行错误

```bash
journalctl -p err -n 100
```

---

### 查看今天的 SSH 登录日志

```bash
journalctl -u sshd --since today
```

---

### 实时监控 Docker

```bash
journalctl -u docker -f
```

---

### 查看服务器刚刚发生的问题

```bash
journalctl --since "10 minutes ago"
```

---

## 与传统日志文件的区别

| 传统方式                | journalctl                       |
| ------------------- | -------------------------------- |
| `/var/log/messages` | 查看所有系统日志                         |
| `/var/log/syslog`   | 使用 `journalctl` 统一查看             |
| `tail -f`           | `journalctl -f`                  |
| `dmesg`             | `journalctl -k`                  |
| 按文件存储               | 按数据库（Journal）存储，可按时间、服务、优先级等快速过滤 |

`journalctl` 的优势在于日志带有结构化字段（如服务名、PID、启动编号、优先级等），可以高效按条件筛选，而无需依赖日志文件路径或复杂的文本搜索。([Linuxize][1])

### 常用命令速查

| 功能         | 命令                                                              |
| ---------- | --------------------------------------------------------------- |
| 查看全部日志     | `journalctl`                                                    |
| 查看最后 100 行 | `journalctl -n 100`                                             |
| 实时查看       | `journalctl -f`                                                 |
| 查看某服务      | `journalctl -u nginx`                                           |
| 实时查看某服务    | `journalctl -u nginx -f`                                        |
| 查看本次启动     | `journalctl -b`                                                 |
| 查看上次启动     | `journalctl -b -1`                                              |
| 查看内核日志     | `journalctl -k`                                                 |
| 查看错误日志     | `journalctl -p err`                                             |
| 查看今天日志     | `journalctl --since today`                                      |
| 查看最近 1 小时  | `journalctl --since "1 hour ago"`                               |
| 搜索关键字      | `journalctl -g failed`                                          |
| 查看日志占用     | `journalctl --disk-usage`                                       |
| 清理日志       | `journalctl --vacuum-size=500M` 或 `journalctl --vacuum-time=7d` |

[1]: https://linuxize.com/post/journalctl-command-in-linux/ "journalctl Command in Linux: Query and Filter System Logs | Linuxize"
