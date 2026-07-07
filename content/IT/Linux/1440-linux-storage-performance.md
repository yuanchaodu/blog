---
title: Linux 存储性能分析
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnnYl
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/144'
---

# Linux 存储性能分析

<img src="images/Linux.svg" width="300">

Linux 存储性能分析，核心不是“看磁盘快不快”，而是回答三个问题：**慢在哪里、谁造成的、为什么会慢**。

Linux 内核会通过 `/proc/diskstats` 和 `/sys/block/<device>/stat` 暴露块设备统计信息，`iostat`、`sar` 等工具主要基于这些数据进行分析。现代 Linux 还可以利用 PSI 判断 I/O 资源不足是否真正导致业务任务停顿。([Linux内核文档][1])

## 一、先理解 Linux 存储 I/O 路径

一次典型的文件访问，大致经过：

```text
应用程序
   ↓
系统调用 read/write
   ↓
文件系统 ext4/xfs
   ↓
Page Cache 页面缓存
   ↓
块设备层
   ↓
设备驱动
   ↓
SSD / NVMe / SAN / 云盘
```

因此，“磁盘慢”不一定真是磁盘问题。例如：

* 应用程序大量随机读写；
* 内存不足导致缓存命中率下降；
* 文件系统出现延迟；
* I/O 队列过深；
* 虚拟机或容器争抢存储；
* SAN 网络存在延迟；
* 设备本身达到性能上限。

可以把存储分析理解为“查堵车”：不能只看终点，还要沿着整条道路逐层检查。

## 二、必须掌握的 5 个指标

| 指标          | 含义           | 主要用途     |
| ----------- | ------------ | -------- |
| IOPS        | 每秒 I/O 次数    | 判断处理能力   |
| Throughput  | 每秒读写数据量      | 判断带宽     |
| Latency     | 单次 I/O 延迟    | 判断业务响应速度 |
| Queue Depth | 等待处理的 I/O 数量 | 判断是否排队   |
| Utilization | 设备忙碌程度       | 判断设备负载   |

其中，**延迟通常最重要**。

例如，两块磁盘都达到 500 MB/s：

```text
磁盘 A：平均延迟 1 ms
磁盘 B：平均延迟 50 ms
```

对数据库和在线业务来说，磁盘 B 的体验会明显更差。

## 三、推荐的分析顺序

我建议使用“**系统 → 设备 → 进程 → I/O 请求 → 内核**”五层分析法。

1. **系统层**：确认是不是真的存在 I/O 压力；
2. **设备层**：确认是哪块磁盘慢；
3. **进程层**：找到谁在读写；
4. **请求层**：判断随机/顺序、读/写、请求大小；
5. **内核层**：复杂问题再使用 eBPF 等工具深入追踪。

这样可以避免一开始就使用复杂工具。

## 四、第一步：判断系统是否真的存在 I/O 问题

先执行：

```bash
vmstat 1
```

重点观察：

```text
wa    CPU 等待 I/O 的时间比例
b     因资源等待而阻塞的任务数量
```

如果 `wa` 持续较高，只能说明 CPU 有时间花在 I/O 等待上，**不能直接证明磁盘已经达到极限**。

再看：

```bash
cat /proc/pressure/io
```

输出类似：

```text
some avg10=2.50 avg60=1.20 avg300=0.50
full avg10=0.30 avg60=0.10 avg300=0.05
```

PSI（Pressure Stall Information，压力停顿信息）衡量资源不足导致任务停顿的时间影响，比单纯看磁盘利用率更接近“业务是否真的受到影响”。`some` 表示至少部分任务因 I/O 停顿，`full` 表示所有非空闲任务都处于停顿状态。([Linux内核文档][2])

这是现代 Linux 性能分析中很有价值的一项指标。

## 五、第二步：找到有问题的磁盘

最常用：

```bash
iostat -xz 1
```

重点关注：

```text
r/s       每秒读请求
w/s       每秒写请求
rkB/s     读取带宽
wkB/s     写入带宽
await     平均 I/O 延迟
aqu-sz    平均队列长度
%util     设备忙碌程度
```

Linux 内核的这些设备统计数据来自块设备层，底层数据可在 `/proc/diskstats` 和 `/sys/block/<device>/stat` 中查看。([Linux内核文档][1])

判断时不要只盯着 `%util`。

例如：

```text
await = 50 ms
aqu-sz = 20
%util  = 99%
```

这通常说明请求正在明显排队。

但对 NVMe 来说：

```text
%util = 100%
```

并不一定代表设备已经完全饱和。现代 NVMe 支持并行队列，因此还要结合**延迟、队列深度和实际业务性能**判断。

## 六、第三步：找到是谁在产生 I/O

可以使用：

```bash
pidstat -d 1
```

重点看：

```text
kB_rd/s
kB_wr/s
iodelay
```

也可以使用：

```bash
iotop
```

分析思路是：

```text
哪块磁盘慢？
      ↓
哪个进程在访问？
      ↓
读还是写？
      ↓
持续发生还是周期发生？
```

例如发现：

```text
mysqld     大量随机读
backup.sh  大量顺序写
```

这两种情况的处理方向完全不同。

## 七、第四步：判断 I/O 类型

存储性能高度依赖访问模式。

主要分析：

```text
顺序读 / 随机读
顺序写 / 随机写
小块 I/O / 大块 I/O
同步 I/O / 异步 I/O
直接 I/O / 缓存 I/O
```

例如：

```text
1 MB 顺序读
```

主要考验带宽。

而：

```text
4 KB 随机读
```

主要考验 IOPS 和延迟。

所以，不能简单地说：

> 这块 SSD 能达到 3 GB/s，因此性能很好。

如果数据库主要执行 4 KB 随机访问，3 GB/s 的顺序带宽参考价值可能很低。

## 八、第五步：深入分析复杂问题

当 `iostat` 和 `pidstat` 只能发现“慢”，但不能解释“为什么慢”时，可以进入内核级分析。

常见工具包括：

```text
perf
blktrace
bpftrace
BCC
```

eBPF 特别适合回答：

```text
哪些进程的 I/O 延迟最高？
哪些 I/O 请求超过 100 ms？
延迟发生在文件系统还是块设备层？
I/O 请求大小如何分布？
```

这比只看平均值更有效。

例如平均延迟：

```text
平均值：5 ms
```

看起来正常，但真实分布可能是：

```text
99% 请求：1 ms
1% 请求：400 ms
```

这 1% 的慢请求，可能正是业务偶尔卡顿的原因。

## 九、一套实用的排查命令

日常排查可以按下面顺序执行：

```bash
# 1. 系统整体状态
uptime
vmstat 1

# 2. I/O 压力
cat /proc/pressure/io

# 3. 磁盘设备
iostat -xz 1

# 4. 进程 I/O
pidstat -d 1

# 5. 实时进程
iotop

# 6. 内存和缓存
free -h

# 7. 文件系统容量
df -h
df -i

# 8. 设备错误
dmesg -T
```

建议把分析逻辑记成一句话：

```text
先看业务是否受影响
→ 再看哪块设备异常
→ 再找哪个进程造成
→ 再分析 I/O 模式
→ 最后进入内核追踪
```

## 十、企业环境中更值得关注的方向

对于生产系统，我不建议只监控磁盘利用率。更完整的监控指标应包括：

```text
业务响应时间
I/O 延迟 P95 / P99
IOPS
读写带宽
队列深度
PSI I/O 压力
文件系统空间
inode 使用率
设备错误
```

特别是数据库、虚拟化平台、容器平台和工业应用服务器，**P99 延迟通常比平均延迟更有价值**。平均值像“全班平均成绩”，可能掩盖少数非常严重的问题。

从当前 Linux 的发展方向看，存储分析正在从传统的“设备利用率监控”，转向“业务停顿 + 延迟分布 + eBPF 动态追踪”。PSI 已经可以直接量化资源压力对任务执行造成的影响，而 cgroup v2 还能按工作负载观察和控制 I/O 资源。([Linux内核文档][2])

如果你准备系统学习这个主题，我建议下一步按 **“iostat 指标详解 → 实际案例分析 → eBPF 存储性能分析 → 企业监控方案”** 的顺序继续。

[1]: https://docs.kernel.org/admin-guide/iostats.html "I/O statistics fields"
[2]: https://docs.kernel.org/accounting/psi.html "PSI - Pressure Stall Information"
