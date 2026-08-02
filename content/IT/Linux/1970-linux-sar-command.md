---
title: Linux sar 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoM-8
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/197'
---

<img src="images/Linux.svg" width="300">

# Linux sar 命令

`sar`（**System Activity Reporter**）是 Linux 中最常用的系统性能监控工具之一，属于 **sysstat** 工具集。它可以查看 CPU、内存、磁盘、网络、I/O、进程等系统资源的使用情况，既可以查看**当前实时数据**，也可以查看**历史性能数据**，是排查性能问题的重要工具。

---

# 一、sar 的工作原理

sar 并不是自己采集数据，而是由 **sadc（System Activity Data Collector）** 定时采集系统信息。

整个流程如下：

```
Linux 内核
      │
      ▼
    sadc（采集）
      │
      ▼
/var/log/sa/sa01
/var/log/sa/sa02
......
      │
      ▼
     sar（读取分析）
```

如果系统没有安装：

```bash
# RHEL/CentOS
yum install sysstat

# Rocky/Alma
dnf install sysstat

# Ubuntu/Debian
apt install sysstat
```

查看版本：

```bash
sar -V
```

---

# 二、sar 基本语法

```bash
sar [选项] [间隔] [次数]
```

例如：

```bash
sar 1 5
```

表示：

* 每隔 1 秒采集一次
* 共采集 5 次

输出类似：

```
Linux 5.14

12:01:01 AM CPU %user %system %idle
12:01:02 AM all  2.31    1.25   96.12
12:01:03 AM all  1.20    0.80   97.45
...
Average:      1.75   1.02  96.78
```

---

# 三、最常用的 sar 参数

## 1、CPU 使用率（默认）

```bash
sar -u
```

或

```bash
sar
```

输出：

```
%user
%system
%iowait
%steal
%idle
```

含义：

| 字段      | 说明       |
| ------- | -------- |
| %user   | 用户态CPU   |
| %system | 内核态CPU   |
| %nice   | nice进程   |
| %iowait | 等待IO     |
| %steal  | 虚拟机被宿主抢占 |
| %idle   | 空闲CPU    |

例如：

```
CPU %user %system %iowait %idle
all 20.3 4.5 1.2 74.0
```

说明：

* 用户程序占20%
* 内核4%
* IO等待1%
* CPU还有74%空闲

---

## 2、查看每个CPU

```bash
sar -P ALL
```

输出：

```
CPU %user %system %idle

0
1
2
3
```

可以发现：

* 是否某个CPU负载异常
* 是否CPU绑定不均衡

查看 CPU0：

```bash
sar -P 0
```

---

## 3、内存使用

```bash
sar -r
```

输出：

```
kbmemfree
kbmemused
%memused
kbbuffers
kbcached
```

例如：

```
kbmemfree   123456
kbmemused   543210
%memused    81%
```

---

## 4、Swap

```bash
sar -S
```

输出：

```
kbswpfree
kbswpused
%swpused
```

如果 Swap 使用持续增长，可能意味着内存压力较大。

---

## 5、分页情况

```bash
sar -B
```

主要关注：

```
pgpgin/s
pgpgout/s
fault/s
majflt/s
```

尤其：

```
majflt/s
```

如果持续较高，说明发生大量主缺页（Major Page Fault），可能导致性能下降。

---

## 6、磁盘IO

```bash
sar -b
```

输出：

```
tps
rtps
wtps
bread/s
bwrtn/s
```

字段说明：

| 字段      | 说明     |
| ------- | ------ |
| tps     | 每秒IO次数 |
| rtps    | 读IO    |
| wtps    | 写IO    |
| bread/s | 读块数    |
| bwrtn/s | 写块数    |

---

## 7、磁盘设备

```bash
sar -d
```

输出：

```
DEV
tps
rkB/s
wkB/s
await
aqu-sz
%util
```

重点关注：

```
await
%util
```

例如：

```
await 40ms
%util 98%
```

说明：

磁盘已经接近饱和。

---

## 8、网络

查看网卡：

```bash
sar -n DEV
```

输出：

```
IFACE
rxkB/s
txkB/s
rxpck/s
txpck/s
```

例如：

```
eth0
```

查看 TCP：

```bash
sar -n TCP
```

查看 TCP 详细统计：

```bash
sar -n ETCP
```

查看网络错误：

```bash
sar -n EDEV
```

---

## 9、上下文切换

```bash
sar -w
```

输出：

```
proc/s
cswch/s
```

字段：

| 字段      | 说明    |
| ------- | ----- |
| proc/s  | 创建进程数 |
| cswch/s | 上下文切换 |

如果：

```
cswch/s
```

非常高，可能：

* 线程太多
* 锁竞争严重
* CPU调度压力大

---

## 10、队列长度

```bash
sar -q
```

输出：

```
runq-sz
plist-sz
ldavg-1
ldavg-5
ldavg-15
```

对应：

```
uptime
```

中的 Load Average。

---

## 11、进程统计

```bash
sar -v
```

包括：

```
inode
file
pty
```

主要用于查看：

* inode 是否耗尽
* 文件句柄是否耗尽

---

# 四、查看历史数据

sar 最大优势之一是可以分析历史性能。

例如查看今天：

```bash
sar -u -f /var/log/sa/sa02
```

查看昨天：

```bash
sar -u -f /var/log/sa/sa01
```

查看指定时间：

```bash
sar -u -s 10:00:00 -e 11:00:00
```

表示：

```
10点~11点
```

查看 CPU：

```bash
sar -u -f /var/log/sa/sa02 -s 09:00:00 -e 09:30:00
```

---

# 五、常用排查命令

## CPU 高

```bash
sar -u 1 10
```

查看每核：

```bash
sar -P ALL 1 5
```

---

## 内存不足

```bash
sar -r 1 5
```

查看 Swap：

```bash
sar -S
```

---

## IO 高

```bash
sar -b
```

磁盘详情：

```bash
sar -d
```

---

## 网络异常

```bash
sar -n DEV
```

TCP：

```bash
sar -n TCP
```

---

## Load 高

```bash
sar -q
```

结合：

```bash
sar -u
sar -d
```

判断：

* CPU 忙？
* IO 忙？
* 进程过多？

---

# 六、常见问题分析

| 现象                       | 可能原因      | 建议进一步排查                |
| ------------------------ | --------- | ---------------------- |
| `%idle` 很低               | CPU 繁忙    | `top`、`pidstat`、`perf` |
| `%iowait` 很高             | 磁盘 I/O 等待 | `iostat`、`iotop`       |
| `%util` 接近 100%          | 磁盘饱和      | 检查磁盘性能、优化 I/O          |
| `await` 很高               | I/O 响应慢   | 分析存储、队列长度              |
| `runq-sz` 大于 CPU 核数      | CPU 调度压力大 | 查看高 CPU 进程             |
| `cswch/s` 很高             | 频繁上下文切换   | 检查线程数、锁竞争              |
| `%memused` 持续很高且 Swap 增长 | 内存压力大     | `free`、`vmstat`、`smem` |
| `rxerr/s`、`txerr/s` 增长   | 网络异常      | 检查网卡、交换机、驱动            |

---

# 七、与其他性能工具的配合

| 工具             | 主要用途             | 特点           |
| -------------- | ---------------- | ------------ |
| `sar`          | 查看系统整体性能及历史数据    | 适合趋势分析和故障回溯  |
| `top` / `htop` | 实时查看进程资源占用       | 定位高 CPU、内存进程 |
| `vmstat`       | CPU、内存、I/O 综合状态  | 适合快速观察系统负载   |
| `iostat`       | 磁盘 I/O 性能        | 分析磁盘吞吐和延迟    |
| `pidstat`      | 按进程统计 CPU、内存、I/O | 定位资源消耗最大的进程  |
| `mpstat`       | 各 CPU 核心使用率      | 分析多核负载是否均衡   |
| `free`         | 内存和 Swap 使用情况    | 快速查看内存状态     |
| `ss`           | 网络连接和 Socket 状态  | 排查网络连接问题     |

## 总结

`sar` 是 Linux 性能分析中功能最全面的工具之一，其优势在于**支持历史数据回放**，非常适合定位“某个时间段为什么系统变慢”这类问题。实际运维中，建议重点掌握以下命令：

```bash
# CPU
sar -u
sar -P ALL

# 内存
sar -r
sar -S

# 磁盘
sar -b
sar -d

# 网络
sar -n DEV
sar -n TCP

# 系统负载
sar -q

# 上下文切换
sar -w

# 查看历史数据
sar -u -f /var/log/sa/sa$(date +%d)
```

结合 `top`、`pidstat`、`iostat` 等工具，可以从系统整体到具体进程逐步缩小范围，快速定位性能瓶颈。
