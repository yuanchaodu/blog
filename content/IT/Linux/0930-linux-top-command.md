---
title: Linux top 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnLkL
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/93'
---

#  Linux top 命令

<img src="images/Linux.svg" width="300">

`top` 是 Linux 中最常用的系统监控命令之一，用于实时查看系统运行状态，包括 CPU、内存、进程、负载等信息。它相当于 Windows 的“任务管理器”。

## 一、基本用法

```bash
top
```

执行后会进入实时监控界面，默认每隔 3 秒刷新一次。

退出：

```bash
q
```

---

## 二、界面组成

### 1. 系统概要信息（顶部）

例如：

```text
top - 14:30:25 up 10 days,  2:35,  2 users,  load average: 0.52, 0.68, 0.75
```

含义：

| 字段           | 说明               |
| ------------ | ---------------- |
| 14:30:25     | 当前时间             |
| up 10 days   | 系统运行时间           |
| 2 users      | 当前登录用户数          |
| load average | 1分钟、5分钟、15分钟平均负载 |

### Load Average（系统负载）

例如：

```text
load average: 0.52, 0.68, 0.75
```

如果服务器有：

* 1核CPU：负载长期超过1需关注
* 4核CPU：负载长期超过4需关注
* 8核CPU：负载长期超过8需关注

简单理解：

> 负载表示等待CPU处理的任务数量。

---

### 2. 任务统计

例如：

```text
Tasks: 256 total, 1 running, 255 sleeping
```

含义：

| 字段       | 说明     |
| -------- | ------ |
| total    | 总进程数   |
| running  | 运行中的进程 |
| sleeping | 休眠进程   |
| stopped  | 暂停进程   |
| zombie   | 僵尸进程   |

---

### 3. CPU使用率

例如：

```text
%Cpu(s): 5.0 us, 2.0 sy, 0.0 ni, 92.0 id, 1.0 wa
```

主要字段：

| 字段 | 含义           |
| -- | ------------ |
| us | 用户进程占用CPU    |
| sy | 内核占用CPU      |
| ni | 调整优先级进程占用CPU |
| id | 空闲CPU        |
| wa | 等待IO时间       |
| hi | 硬件中断         |
| si | 软件中断         |

重点关注：

* `id` 很低（<10%）说明CPU繁忙
* `wa` 很高说明磁盘IO可能存在瓶颈

---

### 4. 内存信息

例如：

```text
MiB Mem : 15933.5 total,  1234.8 free,  8000.0 used,  6698.7 buff/cache
```

字段：

| 字段         | 说明   |
| ---------- | ---- |
| total      | 总内存  |
| free       | 空闲内存 |
| used       | 已使用  |
| buff/cache | 缓存   |

Linux 会尽可能利用空闲内存做缓存，因此：

> free 很少不一定代表内存不足。

更应关注：

```text
available
```

可用内存。

---

### 5. Swap 信息

例如：

```text
MiB Swap: 2048.0 total, 2048.0 free, 0.0 used
```

Swap 是交换分区。

如果：

```text
Swap used 持续增长
```

说明物理内存可能不足。

---

## 三、进程列表

下面部分是进程明细：

```text
PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
```

常见字段：

| 字段      | 说明      |
| ------- | ------- |
| PID     | 进程ID    |
| USER    | 所属用户    |
| PR      | 优先级     |
| NI      | Nice值   |
| VIRT    | 虚拟内存    |
| RES     | 实际占用内存  |
| SHR     | 共享内存    |
| S       | 状态      |
| %CPU    | CPU占用率  |
| %MEM    | 内存占用率   |
| TIME+   | 累计CPU时间 |
| COMMAND | 进程名称    |

---

## 四、常用交互快捷键

运行 `top` 后可直接按键操作：

### 按 CPU 排序

```text
P
```

（大写P）

查看最耗CPU的进程。

---

### 按内存排序

```text
M
```

（大写M）

查看最耗内存的进程。

---

### 按运行时间排序

```text
T
```

查看CPU累计时间最长的进程。

---

### 查看CPU核心情况

```text
1
```

显示：

```text
CPU0
CPU1
CPU2
CPU3
```

适合排查某个核心是否被打满。

---

### 查找进程

```text
L
```

输入关键字搜索。

---

### 杀死进程

```text
k
```

输入：

```text
PID
```

然后输入：

```text
15
```

表示：

```text
SIGTERM
```

若无法结束：

```text
9
```

强制结束。

---

## 五、常用启动参数

### 查看指定用户进程

```bash
top -u oracle
```

---

### 查看指定 PID

```bash
top -p 1234
```

多个进程：

```bash
top -p 1234,5678
```

---

### 设置刷新时间

每秒刷新：

```bash
top -d 1
```

每5秒刷新：

```bash
top -d 5
```

---

### 批处理模式

适用于脚本采集：

```bash
top -b -n 1
```

输出一次后退出。

例如：

```bash
top -b -n 1 > top.log
```

---

## 六、生产环境常用排查命令

### 查看最耗CPU进程

```bash
top
```

然后按：

```text
P
```

---

### 查看最耗内存进程

```bash
top
```

然后按：

```text
M
```

---

### 查看某Java进程

```bash
top -p $(pgrep java)
```

---

### 查看某个CPU核心是否打满

```bash
top
```

按：

```text
1
```

如果发现：

```text
CPU3 100%
```

说明某线程可能存在问题。

进一步定位：

```bash
top -Hp PID
```

查看线程级CPU使用情况。

---

## 七、企业运维中的典型分析思路

当用户反馈“系统变慢”时，通常按以下顺序查看：

```text
top
 ↓
CPU是否高
 ↓
内存是否不足
 ↓
load average是否异常
 ↓
wa(IO等待)是否过高
 ↓
定位具体进程
 ↓
定位线程
 ↓
分析日志
```

对于生产系统（如 MES、ERP、数据库、数字化工厂平台），`top` 往往是故障排查的第一步，可以快速判断问题主要出在：

* CPU瓶颈
* 内存瓶颈
* 磁盘IO瓶颈
* 进程异常
* 线程异常
* 系统负载过高

如果你的环境是 CentOS、Rocky Linux、Red Hat 或 Ubuntu 服务器，我还可以进一步介绍 `top` 与 `htop`、`vmstat`、`iostat`、`pidstat` 联合使用的性能分析方法，以及如何通过 `top -Hp` 分析 Java、Oracle、MySQL 等生产系统的高 CPU 问题。
