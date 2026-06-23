---
title: Linux 调度机制
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnT9G
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/110'
---

# Linux 调度机制

<img src="images/Linux.svg" width="300">

Linux 调度器（Scheduler）是操作系统内核的重要组成部分，它负责决定：

> **哪个进程（或线程）在什么时间使用CPU，以及使用多久。**

可以把CPU想象成一条高速公路，而进程就像等待通行的车辆。调度器的任务就是合理安排车辆通行，既不能让某辆车一直占道，也不能让其他车辆长期等待。

---

# 一、为什么需要调度器

现代Linux系统通常同时运行很多程序：

* 浏览器
* 数据库
* SSH服务
* Docker容器
* Java应用
* 系统服务

但CPU核心数量有限。

例如：

| 资源    | 数量  |
| ----- | --- |
| CPU核心 | 8   |
| 运行线程  | 500 |

显然：

> 500个线程不可能同时运行。

因此必须由调度器决定：

* 谁先运行
* 谁后运行
* 谁运行更久
* 谁优先级更高

---

# 二、Linux进程状态

调度器主要管理可运行（Runnable）的进程。

常见状态：

| 状态       | 含义         |
| -------- | ---------- |
| Running  | 正在CPU上运行   |
| Runnable | 可以运行，等待CPU |
| Sleeping | 等待事件       |
| Stopped  | 暂停         |
| Zombie   | 僵尸进程       |

查看：

```bash
ps -eo pid,stat,comm
```

例如：

```text
PID STAT COMMAND
100 R    nginx
101 S    mysqld
102 D    java
```

其中：

* R = 运行或等待运行
* S = 睡眠
* D = 不可中断睡眠

调度器主要关注：

```text
Running
Runnable
```

---

# 三、Linux调度策略分类

Linux有多个调度类（Scheduling Class）。

优先级从高到低：

```text
SCHED_DEADLINE
    ↓
SCHED_FIFO
    ↓
SCHED_RR
    ↓
SCHED_NORMAL
```

其中：

* SCHED_NORMAL（普通进程）
* SCHED_FIFO（实时）
* SCHED_RR（实时轮转）
* SCHED_DEADLINE（截止时间调度）

SCHED_DEADLINE优先级最高。([维基百科][1])

---

# 四、普通进程调度（最常见）

绝大多数程序：

```text
nginx
mysql
java
python
docker
```

都属于：

```text
SCHED_NORMAL
```

---

## 1 Linux 6.6之前：CFS

CFS全称：

**Completely Fair Scheduler**

即：

> 完全公平调度器

Linux从2.6.23开始长期使用CFS作为默认调度器。([维基百科][2])

### 核心思想

假设有：

```text
A
B
C
```

三个进程。

理想情况下：

```text
A 33%
B 33%
C 33%
```

CPU时间平均分配。

---

## vruntime

CFS引入一个关键概念：

```text
vruntime
```

虚拟运行时间。

例如：

| 进程 | vruntime |
| -- | -------- |
| A  | 10       |
| B  | 20       |
| C  | 30       |

调度器会选择：

```text
A
```

因为它运行得最少。

简单理解：

> 谁欠CPU时间最多，谁先运行。([LWN.net][3])

---

## 红黑树

CFS使用：

```text
Red-Black Tree
```

保存所有可运行进程。

结构：

```text
          30
         /  \
       20    50
      /
    10
```

最左边节点：

```text
10
```

即最小vruntime。

直接选它运行。

复杂度：

```text
O(logN)
```

效率非常高。([维基百科][2])

---

# 五、Linux 6.6以后：EEVDF

从Linux 6.6开始，默认调度器逐步由：

```text
CFS
↓
EEVDF
```

替代。([Linux内核文档][4])

EEVDF全称：

**Earliest Eligible Virtual Deadline First**

即：

> 最早可执行虚拟截止时间优先

---

## 为什么替换CFS

CFS主要追求：

```text
公平
```

但有时会影响：

```text
响应速度
```

例如：

* 视频会议
* GUI界面
* 游戏
* 音频处理

这些任务更关注：

```text
低延迟
```

而不是绝对公平。

因此引入EEVDF。([LWN.net][3])

---

## EEVDF核心思想

每个任务维护：

* vruntime
* lag（欠账）
* Virtual Deadline（虚拟截止时间）

调度器首先选择：

```text
lag >= 0
```

即真正需要CPU时间的任务。

然后选择：

```text
最早到期
```

的任务运行。([Linux内核文档][4])

---

### 举例

| 任务 | 虚拟截止时间 |
| -- | ------ |
| A  | 100    |
| B  | 120    |
| C  | 150    |

调度器选择：

```text
A
```

因为它最早到期。

这样：

* 保持公平
* 提高交互响应

两者兼顾。([Linux内核文档][4])

---

# 六、实时调度

对于工业控制、通信系统等场景：

普通调度器不够。

Linux提供：

```text
SCHED_FIFO
SCHED_RR
```

---

## SCHED_FIFO

先进先出。

特点：

```text
高优先级任务
一直运行
直到：
    主动让出CPU
    阻塞
    被更高优先级抢占
```

例如：

```text
优先级90
```

只要不阻塞：

```text
一直运行
```

---

### 查看

```bash
chrt -p PID
```

---

## 设置

```bash
sudo chrt -f 90 ./app
```

表示：

```text
FIFO
优先级90
```

---

# 七、SCHED_RR

RR：

```text
Round Robin
```

时间片轮转。

假设：

```text
A
B
C
```

优先级相同。

运行顺序：

```text
A→B→C→A→B→C
```

每人一个时间片。

---

# 八、SCHED_DEADLINE

Linux最强大的实时调度策略。

基于：

```text
EDF
(Earliest Deadline First)
```

算法。([维基百科][5])

---

例如：

| 任务 | 截止时间 |
| -- | ---- |
| A  | 10ms |
| B  | 20ms |
| C  | 50ms |

调度器优先执行：

```text
A
```

因为最紧急。

---

适用于：

* 工业控制
* 自动驾驶
* 通信基站
* 航空航天

---

# 九、多核CPU调度

现代服务器：

```text
16核
32核
64核
128核
```

很常见。

Linux采用：

```text
Per-CPU Runqueue
```

每个CPU一个运行队列。([维基百科][2])

例如：

```text
CPU0
 └─ rq0

CPU1
 └─ rq1

CPU2
 └─ rq2
```

这样减少锁竞争。

---

## Load Balance

如果：

```text
CPU0
100个任务

CPU1
5个任务
```

Linux会迁移任务：

```text
CPU0 → CPU1
```

实现负载均衡。

---

# 十、上下文切换（Context Switch）

当调度器决定切换任务时：

```text
A -> B
```

需要保存：

```text
寄存器
程序计数器
栈
内存映射
```

然后恢复B的状态。

这称为：

```text
Context Switch
```

---

查看：

```bash
vmstat 1
```

输出：

```text
cs
```

表示每秒上下文切换次数。

---

# 十一、调度相关命令

## 查看CPU调度情况

```bash
top
```

---

## 更详细

```bash
htop
```

---

## 查看实时优先级

```bash
ps -eo pid,pri,ni,cmd
```

---

## 查看调度策略

```bash
chrt -p PID
```

---

## 查看线程

```bash
top -H
```

---

## 查看CPU绑定

```bash
taskset -p PID
```

---

# 十二、企业生产环境关注点

对于制造企业、数字化工厂和MES系统环境，重点通常不是研究调度算法本身，而是关注调度对业务的影响：

### 数据库服务器

关注：

* CPU利用率
* Run Queue长度
* 上下文切换

工具：

```bash
vmstat
sar
top
```

---

### Java应用服务器

关注：

* GC线程抢占CPU
* CPU Affinity
* NUMA

---

### 实时控制系统

关注：

* SCHED_FIFO
* PREEMPT_RT
* CPU隔离

---

### Docker/Kubernetes

关注：

* CPU Shares
* CPU Quota
* Cgroup调度

---

# 调度机制总结图

```text
Linux Scheduler
│
├─ SCHED_NORMAL
│    ├─ CFS（6.6前）
│    └─ EEVDF（6.6后）
│
├─ SCHED_FIFO
│
├─ SCHED_RR
│
└─ SCHED_DEADLINE
```

记住一句话即可：

> Linux调度器的目标是在“公平性、吞吐量、响应速度、实时性”之间找到平衡。普通业务系统主要依赖 EEVDF/CFS，工业控制和通信系统则更多使用 FIFO、RR 或 DEADLINE 等实时调度策略。([Linux内核文档][4])

[1]: https://en.wikipedia.org/wiki/Linux_kernel "Linux kernel"
[2]: https://en.wikipedia.org/wiki/Completely_Fair_Scheduler "Completely Fair Scheduler"
[3]: https://lwn.net/Articles/925371/ "An EEVDF CPU scheduler for Linux"
[4]: https://docs.kernel.org/scheduler/sched-eevdf.html "EEVDF Scheduler"
[5]: https://en.wikipedia.org/wiki/SCHED_DEADLINE "SCHED DEADLINE"
