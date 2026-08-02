---
title: Linux 故障排查实战
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoM_1
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/200'
---

<img src="images/Linux.svg" width="300">

# Linux 故障排查实战

Linux 故障排查是一项综合能力，核心目标是**快速定位问题、分析根因、恢复业务、防止再次发生**。在企业环境中，80%以上的故障都集中在 CPU、内存、磁盘、网络、进程、服务、权限、日志这几个方面。

下面按照企业生产环境的思路，介绍一套完整的 Linux 故障排查实战方法。

---

# 一、故障排查总体思路

建议遵循下面这个流程，而不是想到什么查什么。

```
用户反馈
    │
    ▼
确认故障现象
    │
    ▼
确定影响范围
    │
    ▼
收集系统信息
    │
    ▼
分析日志
    │
    ▼
定位根因
    │
    ▼
恢复服务
    │
    ▼
验证恢复
    │
    ▼
总结复盘
```

可以总结成一句话：

> **先看现象，再看资源，再看日志，最后定位原因。**

---

# 二、第一步：确认系统是否还活着

不要急着登录应用。

先确认：

```
服务器能不能Ping
SSH是否可以登录
```

如果可以登录：

```bash
uptime
```

例如：

```
16:12:11 up 152 days,  2 users,
load average: 18.25 17.30 15.12
```

重点看：

* uptime
* load average

load并不是CPU使用率。

经验值：

```
Load < CPU核数
正常

Load > CPU核数2倍
开始排查

Load > CPU核数5倍
基本有故障
```

例如：

```
8核机器

Load=3
正常

Load=10
较高

Load=35
严重异常
```

---

# 三、CPU异常排查

最常见命令：

```bash
top
```

或者

```bash
htop
```

重点关注：

```
%Cpu(s):
```

例如：

```
90% us
```

说明：

用户程序占CPU。

如果：

```
80% sy
```

说明：

大量系统调用。

如果：

```
wa很高
```

例如：

```
wa 60%
```

CPU其实没忙。

而是在：

等待磁盘。

这是很多人误判CPU的问题。

---

继续查看：

```bash
ps -ef
```

或者：

```bash
top
```

按：

```
P
```

按照CPU排序。

例如：

```
java
320%

python
180%

mysqld
120%
```

继续：

```bash
pidstat -p PID 1
```

实时观察。

---

# 四、内存不足

查看：

```bash
free -h
```

例如：

```
total 64G

used 62G

free 500M
```

不要马上认为：

内存不足。

Linux大量使用：

```
buff/cache
```

真正看：

```
available
```

例如：

```
available

30G
```

说明：

其实还有30G可用。

---

继续：

```
top
```

按：

```
M
```

按内存排序。

例如：

```
Java

Oracle

Redis
```

再查看：

```bash
pmap PID
```

或者：

```bash
cat /proc/PID/status
```

重点：

```
VmRSS

VmSize
```

---

查看OOM：

```bash
dmesg | grep -i oom
```

或者：

```bash
journalctl -k | grep -i oom
```

例如：

```
Out of memory

Killed process
```

说明：

OOM Killer已经杀掉程序。

---

# 五、磁盘故障

查看：

```bash
df -h
```

例如：

```
Filesystem

Use%

/

98%
```

如果：

```
100%
```

很多程序：

都会失败。

例如：

```
MySQL

Docker

Oracle

日志写入
```

都会异常。

---

继续查看：

```bash
du -sh /*
```

或者：

```bash
du -xh / | sort -h
```

找大目录。

继续：

```bash
find / -size +1G
```

找大文件。

---

查看inode：

```bash
df -i
```

很多人忽略。

例如：

```
Use%

100%
```

即使：

```
磁盘还有100G
```

文件仍然不能创建。

---

# 六、IO性能问题

查看：

```bash
iostat -x 1
```

重点：

```
%util
await
svctm
```

例如：

```
util

100%
```

说明：

磁盘满负荷。

如果：

```
await

300ms
```

说明：

IO等待严重。

---

查看：

```bash
iotop
```

谁在疯狂写盘。

例如：

```
mysqld

backup

tar

rsync
```

---

# 七、网络故障

查看：

```bash
ip addr
```

确认：

IP是否存在。

---

查看：

```bash
ip route
```

默认网关。

---

查看：

```bash
ping
```

测试：

```
网关

DNS

目标服务器
```

---

查看：

```bash
ss -tulnp
```

替代：

```
netstat
```

例如：

```
LISTEN

80

443

3306
```

确认：

服务是否监听。

---

继续：

```bash
ss -s
```

查看连接数。

例如：

```
ESTAB

TIME_WAIT
```

如果：

```
TIME_WAIT

几十万
```

可能：

端口耗尽。

---

抓包：

```bash
tcpdump
```

例如：

```bash
tcpdump -i eth0 port 80
```

查看：

```
有没有请求

有没有返回
```

---

# 八、服务启动失败

查看：

```bash
systemctl status nginx
```

如果失败：

继续：

```bash
journalctl -u nginx
```

例如：

```
Permission denied

Address already in use

No such file
```

定位非常快。

---

查看：

```bash
systemctl list-units --failed
```

所有失败服务。

---

# 九、日志分析

Linux排查最重要：

日志。

系统日志：

```
/var/log/messages

/var/log/syslog
```

Debian：

```
syslog
```

CentOS：

```
messages
```

内核：

```
dmesg
```

Systemd：

```bash
journalctl
```

实时：

```bash
journalctl -f
```

相当于：

```
tail -f
```

应用：

```
nginx

mysql

tomcat

docker
```

分别查看。

---

# 十、文件描述符耗尽

查看：

```bash
ulimit -n
```

系统：

```bash
cat /proc/sys/fs/file-max
```

查看：

```bash
lsof
```

例如：

```
lsof | wc -l
```

查看：

某进程：

```bash
lsof -p PID
```

如果：

几十万个FD。

说明：

程序泄漏。

---

# 十一、进程假死

查看：

```bash
ps -ef
```

状态：

```
R

S

D

Z
```

重点：

```
D
```

不可中断睡眠。

通常：

```
IO卡死
```

如果：

```
Z
```

僵尸进程。

继续：

```bash
pstree -p
```

找到父进程。

---

# 十二、企业常见故障案例

## 案例1：CPU飙高

现象：

```
Load 40
```

步骤：

```
top

↓

发现Java 800%

↓

jstack

↓

线程死循环

↓

重启服务

↓

恢复
```

---

## 案例2：磁盘满

```
df -h
```

发现：

```
/
100%
```

继续：

```
du -sh /*
```

发现：

```
/var/log

500G
```

原因：

日志未轮转。

处理：

```
logrotate

清理历史日志

恢复
```

---

## 案例3：SSH无法登录

Ping正常。

排查：

```
systemctl status sshd

↓

journalctl -u sshd

↓

磁盘100%

↓

sshd无法写日志

↓

清理磁盘

↓

恢复
```

---

## 案例4：MySQL响应慢

```
top
```

CPU正常。

```
free

正常
```

```
iostat

await=500ms
```

说明：

不是数据库。

而是：

磁盘IO瓶颈。

---

# 十三、推荐的排查工具

| 工具         | 用途                  |
| ---------- | ------------------- |
| top / htop | CPU、内存、进程           |
| vmstat     | CPU、内存、上下文切换        |
| iostat     | 磁盘 I/O              |
| iotop      | 找出高 I/O 进程          |
| pidstat    | 进程级 CPU、内存、I/O      |
| sar        | 历史性能数据（需启用 sysstat） |
| free       | 内存使用                |
| df / du    | 磁盘空间                |
| ss         | 网络连接、监听端口           |
| lsof       | 打开的文件、网络连接          |
| tcpdump    | 网络抓包                |
| journalctl | systemd 日志          |
| dmesg      | 内核日志                |
| strace     | 跟踪系统调用              |
| perf       | CPU 性能分析            |
| bpftrace   | eBPF 动态追踪，适合高级性能分析  |

---

# 十四、一张 Linux 故障排查速查图

```text
服务器异常
│
├── 登录失败
│   ├── ping
│   ├── ssh
│   ├── systemctl status sshd
│   └── journalctl
│
├── 系统变慢
│   ├── uptime
│   ├── top
│   ├── vmstat
│   ├── iostat
│   └── pidstat
│
├── CPU 高
│   ├── top
│   ├── ps
│   ├── pidstat
│   └── perf / strace
│
├── 内存高
│   ├── free
│   ├── top(M)
│   ├── pmap
│   └── dmesg(OOM)
│
├── 磁盘满
│   ├── df -h
│   ├── df -i
│   ├── du -sh
│   └── find 大文件
│
├── 网络异常
│   ├── ip addr
│   ├── ip route
│   ├── ss
│   ├── ping
│   └── tcpdump
│
└── 服务异常
    ├── systemctl status
    ├── journalctl -u
    ├── 日志
    └── lsof
```

## 实战建议

对于生产环境，建议掌握一套由浅入深的排查顺序：

1. **先确认故障现象**：是否可登录、是否影响全部用户、是否可复现。
2. **再检查系统资源**：`uptime`、`top`、`free -h`、`df -h`、`vmstat 1`、`iostat -x 1`。
3. **查看日志**：优先检查 `journalctl`、内核日志 (`dmesg`) 和应用日志，很多故障都能直接定位。
4. **定位到具体进程**：结合 `pidstat`、`lsof`、`strace`、`perf` 等工具深入分析。
5. **恢复后复盘**：记录故障时间线、根因、处理过程和预防措施，例如增加监控告警、配置日志轮转、优化资源限制等。

对于企业运维人员而言，与其记忆大量命令，不如建立一套固定的排查流程。按照“**现象 → 资源 → 日志 → 进程 → 根因 → 恢复**”的思路，能够覆盖绝大多数 Linux 生产环境故障，并显著提升定位效率。
