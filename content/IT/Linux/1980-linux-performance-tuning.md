---
title: Linux 性能优化
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoM_Q
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/198'
---

<img src="images/Linux.svg" width="300">

# Linux 性能优化

Linux 性能优化是一个系统性工作，不是简单地修改几个参数。通常需要遵循一个原则：**先监控，再定位，最后优化**。盲目调整内核参数，反而可能降低系统稳定性。

可以把 Linux 性能优化分为六个方面：CPU、内存、磁盘、网络、文件系统以及应用程序。

---

# 一、性能优化流程

建议按照下面的流程进行。

```text
性能下降
     │
     ▼
监控系统资源
     │
     ▼
定位瓶颈(CPU/内存/IO/网络)
     │
     ▼
分析具体进程
     │
     ▼
调整配置或优化程序
     │
     ▼
持续监控
```

Linux 有一句比较经典的话：

> **80%的性能问题，不是Linux内核，而是应用程序设计问题。**

因此首先要判断瓶颈到底在哪里。

---

# 二、CPU优化

## 1）查看CPU使用率

常用工具

```bash
top
htop
mpstat
sar -u 1
vmstat 1
```

重点关注

| 指标 | 说明     |
| -- | ------ |
| us | 用户态CPU |
| sy | 内核CPU  |
| id | 空闲CPU  |
| wa | 等待IO   |
| st | 虚拟机被抢占 |

例如

```text
Cpu(s): 80 us, 5 sy, 2 ni, 13 id
```

说明

* CPU几乎已经满载
* 应用程序占用较高

---

## 2）定位高CPU进程

```bash
top
```

或者

```bash
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu
```

查看线程

```bash
top -Hp PID
```

结合

```bash
jstack
perf
gdb
```

进一步分析。

---

## 3）CPU绑定

查看CPU

```bash
lscpu
```

绑定CPU

```bash
taskset
```

例如

```bash
taskset -c 0,1 ./server
```

对于实时业务可以减少CPU迁移。

---

# 三、内存优化

## 查看内存

```bash
free -h
```

```bash
vmstat 1
```

```bash
cat /proc/meminfo
```

重点

* MemAvailable
* Buffers
* Cached
* Swap

很多人看到

```
Memory Used 95%
```

就认为内存不足。

实际上Linux会尽量利用空闲内存做Cache。

真正应该关注

```
Swap是否大量使用
```

以及

```
OOM Killer
```

查看

```bash
dmesg | grep -i oom
```

---

## 查看进程内存

```bash
top
```

```bash
ps aux --sort=-rss
```

或者

```bash
smem
```

---

## 调整Swap

查看

```bash
cat /proc/sys/vm/swappiness
```

一般服务器

```
10~20
```

修改

```bash
sysctl vm.swappiness=10
```

永久

```
/etc/sysctl.conf
```

---

# 四、磁盘IO优化

磁盘慢，是企业服务器最常见瓶颈之一。

常用工具

```bash
iostat -x 1
```

安装

```bash
yum install sysstat
```

重点指标

| 指标    | 意义            |
| ----- | ------------- |
| %util | 磁盘利用率         |
| await | 平均等待时间        |
| svctm | 服务时间（新版本已不推荐） |
| r/s   | 读次数           |
| w/s   | 写次数           |

例如

```
%util=100%
await=200ms
```

说明

磁盘已经成为瓶颈。

---

查看进程IO

```bash
iotop
```

或者

```bash
pidstat -d
```

---

SSD推荐

调度算法

```text
mq-deadline
none
```

查看

```bash
cat /sys/block/sda/queue/scheduler
```

---

# 五、网络优化

查看连接

```bash
ss -s
```

查看端口

```bash
ss -ant
```

查看流量

```bash
iftop
nload
iptraf
```

查看丢包

```bash
sar -n DEV 1
```

---

网络参数

例如

```bash
net.core.somaxconn
```

```bash
net.ipv4.tcp_tw_reuse
```

```bash
net.ipv4.ip_local_port_range
```

修改

```bash
sysctl
```

例如

```bash
sysctl -w net.core.somaxconn=65535
```

---

# 六、文件系统优化

查看挂载

```bash
mount
```

查看空间

```bash
df -h
```

查看inode

```bash
df -i
```

很多时候

```
磁盘还有空间
```

但是

```
inode已经满了
```

系统仍然无法创建文件。

---

查看目录大小

```bash
du -sh *
```

查找大文件

```bash
find / -size +1G
```

---

# 七、系统参数优化

查看

```bash
sysctl -a
```

比较常见

### 文件句柄

查看

```bash
ulimit -n
```

修改

```
/etc/security/limits.conf
```

例如

```
* soft nofile 65535
* hard nofile 65535
```

---

### 内核参数

例如

```text
fs.file-max
vm.max_map_count
kernel.pid_max
```

数据库和 Elasticsearch 常会调整这些参数。

---

# 八、常用性能分析工具

| 工具                | 用途             |
| ----------------- | -------------- |
| top / htop        | 查看系统资源         |
| vmstat            | CPU、内存、IO综合状态  |
| iostat            | 磁盘IO           |
| sar               | 历史性能统计         |
| pidstat           | 按进程分析CPU、内存、IO |
| iotop             | 磁盘IO占用         |
| iftop             | 网络流量           |
| ss                | TCP连接分析        |
| perf              | CPU热点分析        |
| strace            | 系统调用跟踪         |
| lsof              | 打开文件分析         |
| dstat             | 综合监控           |
| atop              | 系统资源长期记录       |
| bpftrace / eBPF   | 内核级性能分析        |
| perf + FlameGraph | 生成火焰图，定位热点函数   |

---

# 九、企业级性能排查思路

建议按以下顺序进行排查，能够覆盖绝大多数 Linux 性能问题：

1. **确认现象**：CPU 高、内存不足、磁盘 IO 高、网络延迟还是负载（Load Average）升高。
2. **系统级检查**：使用 `top`、`vmstat`、`iostat`、`sar` 等工具判断瓶颈类型。
3. **进程级定位**：使用 `ps`、`pidstat`、`iotop`、`ss` 等找出资源消耗最高的进程。
4. **应用级分析**：结合应用日志、`strace`、`perf`、火焰图（Flame Graph）或语言运行时工具（如 Java 的 `jstack`、Go 的 `pprof`）分析热点。
5. **针对性优化**：优化代码、SQL、缓存策略、线程模型，或调整内核参数和硬件配置。
6. **持续监控**：部署监控平台，长期观察趋势，避免问题再次发生。

---

# 十、推荐学习路线

如果希望系统掌握 Linux 性能优化，建议按以下顺序学习：

1. **Linux 基础**：进程、线程、调度器、虚拟内存、文件系统。
2. **性能指标**：CPU、Load Average、内存、IO、网络的含义及相互关系。
3. **常用工具**：`top`、`vmstat`、`iostat`、`sar`、`pidstat`、`perf`、`strace`。
4. **高级分析**：火焰图（Flame Graph）、eBPF、`bpftrace`、`bcc` 工具集。
5. **监控平台**：Prometheus + Node Exporter + Grafana，实现实时监控与告警。
6. **实战演练**：针对 Web 服务、数据库（如 MySQL）、Java 应用、容器（Docker/Kubernetes）等典型场景进行性能诊断和优化。

掌握上述内容后，基本能够应对生产环境中绝大多数 Linux 性能问题，并形成“**监控—诊断—定位—优化—验证**”的完整闭环，而不是依赖经验进行盲目调参。
