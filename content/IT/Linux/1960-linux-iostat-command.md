---
title: Linux iostat 命令
section: IT
category: Linux
---

<img src="images/Linux.svg" width="300">

# Linux iostat 命令

`iostat`（**I/O Statistics**）是 Linux 中用于监控 **CPU 使用情况**和**磁盘 I/O 性能**的常用工具，属于 **sysstat** 工具集。它可以帮助判断系统是否存在磁盘性能瓶颈，是排查 Linux 性能问题最常用的命令之一。

---

# 一、安装 iostat

大多数发行版需要安装 `sysstat` 软件包。

**CentOS / Rocky / RHEL**

```bash
yum install sysstat
```

或

```bash
dnf install sysstat
```

**Ubuntu / Debian**

```bash
apt install sysstat
```

查看版本：

```bash
iostat -V
```

---

# 二、基本语法

```bash
iostat [选项] [间隔时间] [次数]
```

例如：

```bash
iostat
```

输出一次统计信息。

每隔2秒输出一次：

```bash
iostat 2
```

每2秒输出，共输出5次：

```bash
iostat 2 5
```

---

# 三、最常用命令

## 1）查看CPU和磁盘统计

```bash
iostat
```

例如：

```text
Linux 5.14.0

avg-cpu:
 %user %nice %system %iowait %steal %idle
 2.15   0.00    1.23    0.36    0.00 96.26

Device             tps    kB_read/s    kB_wrtn/s
sda              12.34      520.12        80.25
```

CPU部分：

| 字段      | 说明        |
| ------- | --------- |
| %user   | 用户程序占用CPU |
| %system | 内核占用CPU   |
| %iowait | 等待磁盘I/O时间 |
| %idle   | CPU空闲     |

其中：

**%iowait 最值得关注。**

一般经验：

* <5%：正常
* 5%~20%：开始有I/O压力
* > 20%：磁盘可能成为瓶颈
* > 50%：通常存在严重I/O问题（需结合其他指标确认）

---

## 2）查看扩展磁盘信息（最常用）

```bash
iostat -x
```

或者实时查看：

```bash
iostat -x 2
```

这是运维最常用的方式。

例如：

```text
Device:
sda

r/s
w/s
rkB/s
wkB/s
await
svctm
%util
```

---

# 四、重点字段解释

## r/s

每秒读请求数

例如：

```text
r/s = 200
```

表示：

每秒读取200次。

---

## w/s

每秒写请求数。

例如：

```text
w/s = 150
```

表示：

每秒150次写。

---

## rkB/s

每秒读取KB

例如：

```text
rkB/s = 10240
```

表示：

约10MB/s读取。

---

## wkB/s

每秒写KB

例如：

```text
wkB/s = 5120
```

表示：

约5MB/s写入。

---

## await（非常重要）

表示：

**一次I/O平均等待时间（毫秒）**

包括：

> 排队时间 + 服务时间

例如：

```text
await = 2.3
```

表示：

平均一次IO等待2.3ms。

一般经验：

| await   | 说明   |
| ------- | ---- |
| <5ms    | 很好   |
| 5~20ms  | 正常   |
| 20~50ms | 有压力  |
| >50ms   | 明显瓶颈 |
| >100ms  | 严重问题 |

注意：不同存储介质（机械盘、SATA SSD、NVMe SSD、网络存储）正常范围不同，应结合设备类型判断。

---

## avgqu-sz（队列长度）

表示：

平均等待队列长度。

例如：

```text
avgqu-sz = 5
```

说明：

平均有5个IO在排队。

通常：

* 接近0：很好
* 持续增大：I/O堆积

---

## %util（非常重要）

表示：

磁盘设备忙碌时间百分比。

例如：

```text
%util = 95%
```

表示：

磁盘95%的时间都在工作。

一般经验：

| %util  | 说明     |
| ------ | ------ |
| <50%   | 压力小    |
| 50~80% | 正常     |
| 80~90% | 开始繁忙   |
| >90%   | 可能达到瓶颈 |

**注意：** 对于现代 NVMe SSD，`%util` 不一定能准确反映性能是否达到极限，因为设备支持高并发队列，应结合 `await`、`r/s`、`w/s` 等指标综合判断。

---

# 五、常用参数

## -x

扩展统计

```bash
iostat -x
```

最常用。

---

## -d

只看磁盘

```bash
iostat -d
```

输出：

```text
Device
tps
kB_read/s
kB_wrtn/s
```

---

## -c

只看CPU

```bash
iostat -c
```

---

## -m

以 MB/s 显示

```bash
iostat -xm
```

例如：

```text
rkB/s
```

变成：

```text
rMB/s
```

---

## -k

以KB显示

```bash
iostat -xk
```

默认一般就是KB。

---

## -p

查看分区

例如：

```bash
iostat -p sda
```

输出：

```text
sda
sda1
sda2
```

---

## -y

忽略启动以来第一次统计。

例如：

```bash
iostat -xy 2
```

这是生产环境非常推荐的方式。

原因：

第一次数据显示的是

> 系统启动以来平均值

通常参考意义不大。

---

# 六、常用组合

实时查看：

```bash
iostat -x 1
```

每秒刷新一次。

---

忽略第一次：

```bash
iostat -xy 1
```

这是最推荐的实时观察命令。

---

MB显示：

```bash
iostat -xm 2
```

---

查看CPU：

```bash
iostat -c 2
```

---

查看所有磁盘：

```bash
iostat -dx
```

---

# 七、如何判断磁盘是否有瓶颈

重点关注以下几个指标：

| 指标          | 关注点              |
| ----------- | ---------------- |
| %iowait     | CPU 是否在等待磁盘 I/O  |
| await       | I/O 平均等待时间是否持续升高 |
| avgqu-sz    | 是否存在较长的 I/O 排队   |
| %util       | 设备是否长时间处于高负载状态   |
| r/s、w/s     | 读写请求量是否异常增加      |
| rMB/s、wMB/s | 吞吐量是否接近设备能力      |

例如：

```text
Device    r/s   w/s   await  avgqu-sz  %util
sda      500   300    85.2      42.3     99.8
```

分析如下：

* `await=85.2ms`：I/O 响应较慢。
* `avgqu-sz=42.3`：大量请求排队。
* `%util≈100%`：设备长时间处于繁忙状态。
* `r/s`、`w/s` 较高：读写请求较多。

综合来看，这通常表明该磁盘已成为系统性能瓶颈，需要进一步分析具体进程（如结合 `iotop`、`pidstat -d`、`sar -d` 等工具）以及存储设备能力，判断是业务负载正常增长还是异常 I/O 所致。

---

# 八、常见排查组合

在实际运维中，`iostat` 通常与其他工具配合使用：

| 命令                 | 用途                    |
| ------------------ | --------------------- |
| `iostat -xy 1`     | 观察磁盘 I/O 实时状态         |
| `iotop`            | 查看哪些进程产生大量磁盘 I/O      |
| `pidstat -d 1`     | 按进程统计磁盘读写情况           |
| `vmstat 1`         | 观察 CPU、内存、I/O 等整体状态   |
| `sar -d 1`         | 记录并分析历史磁盘 I/O 数据      |
| `dstat`（部分系统需单独安装） | 综合查看 CPU、磁盘、网络等资源使用情况 |

**推荐记住的命令：**

```bash
# 最常用，实时查看扩展磁盘统计
iostat -xy 1

# 以 MB/s 显示
iostat -xmy 1

# 每 2 秒采样，共输出 10 次
iostat -xy 2 10
```

对于日常性能排查，重点关注 **`%iowait`（CPU 等待 I/O）、`await`（I/O 延迟）、`avgqu-sz`（队列长度）和 `%util`（设备忙碌程度）**，并结合业务负载和存储类型进行综合分析，而不是仅凭某一个指标判断是否存在磁盘瓶颈。