---
title: Linux 进程优先级
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnTog
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/109'
---

# Linux 进程优先级

<img src="images/Linux.svg" width="300">

Linux 中的**进程优先级（Process Priority）**用于决定在 CPU 资源有限时，哪个进程能获得更多的处理时间。Linux 采用抢占式多任务调度机制，优先级越高的进程，通常越容易获得 CPU。

## 一、Linux 进程优先级的基本概念

Linux 中主要有两个相关概念：

### 1. Priority（PRI）

这是内核实际调度时使用的优先级。

特点：

* 数值越小，优先级越高。
* 普通用户一般无法直接修改。
* 动态变化，由调度器根据系统运行情况自动调整。

例如：

```bash
top
```

显示：

```text
PID USER   PR NI
1234 root  20  0
```

其中：

* PR：Priority（实际优先级）
* NI：Nice值

---

### 2. Nice值（NI）

Nice 值是用户能够调整的优先级参数。

取值范围：

```text
-20 ～ 19
```

含义：

| Nice值 | 优先级 |
| ----- | --- |
| -20   | 最高  |
| 0     | 默认  |
| 19    | 最低  |

规律：

```text
Nice值越小
→ 优先级越高
→ 获得CPU机会越多
```

例如：

```bash
NI=-20
```

表示：

```text
“请系统优先照顾我”
```

而：

```bash
NI=19
```

表示：

```text
“我不着急，别人先用CPU”
```

---

## 二、查看进程优先级

### 方法1：top

```bash
top
```

输出：

```text
PID USER  PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
2354 root 20  0 ...
```

关注：

```text
PR = Priority
NI = Nice值
```

---

### 方法2：ps

查看指定进程：

```bash
ps -eo pid,user,pri,ni,comm
```

示例：

```text
PID USER PRI NI COMMAND
1   root  19  0 systemd
345 root  20  0 sshd
```

---

## 三、启动时指定优先级

### nice 命令

默认：

```bash
./app
```

等价于：

```bash
nice -n 0 ./app
```

降低优先级：

```bash
nice -n 10 ./app
```

查看：

```bash
ps -o pid,ni,comm
```

结果：

```text
NI=10
```

---

### 提高优先级

需要 root 权限：

```bash
nice -n -10 ./app
```

或者：

```bash
sudo nice -n -10 ./app
```

---

## 四、修改正在运行进程的优先级

### renice

语法：

```bash
renice 优先级 PID
```

例如：

```bash
sudo renice -10 -p 1234
```

表示：

```text
将1234进程的Nice值改为-10
```

查看：

```bash
ps -o pid,ni,comm -p 1234
```

---

### 降低优先级

```bash
renice 15 -p 1234
```

结果：

```text
NI=15
```

---

## 五、实时进程优先级（Real-Time Priority）

除了普通进程外，Linux 还支持实时调度。

常见策略：

| 策略             | 说明      |
| -------------- | ------- |
| SCHED_OTHER    | 默认普通进程  |
| SCHED_FIFO     | 实时先进先出  |
| SCHED_RR       | 实时时间片轮转 |
| SCHED_DEADLINE | 截止时间调度  |

查看：

```bash
chrt -p PID
```

示例：

```text
pid 1234's current scheduling policy: SCHED_FIFO
pid 1234's current scheduling priority: 50
```

---

### 实时优先级范围

```text
1～99
```

特点：

* 99 最高
* 1 最低
* 实时进程优先级高于普通进程

例如：

```text
实时优先级50
>
普通进程优先级
```

---

### 启动实时进程

```bash
sudo chrt -f 50 ./app
```

表示：

```text
FIFO策略
优先级50
```

查看：

```bash
ps -eo pid,class,rtprio,ni,comm
```

结果：

```text
PID CLS RTPRIO NI COMMAND
1234 FF 50     - app
```

其中：

* FF：FIFO
* RR：Round Robin
* TS：普通进程

---

## 六、实际生产环境中的应用

### 场景1：数据库

例如：

* MySQL
* PostgreSQL

可以适当提高优先级：

```bash
renice -5 -p mysql_pid
```

提高响应速度。

---

### 场景2：备份任务

例如：

```bash
tar
rsync
backup.sh
```

设置低优先级：

```bash
nice -n 19 rsync ...
```

避免影响生产业务。

---

### 场景3：工业控制系统

例如：

* PLC通讯程序
* OPC UA服务
* 实时采集程序

可采用：

```bash
SCHED_FIFO
SCHED_RR
```

确保采集任务及时执行。

---

## 七、优先级与CPU占用率的关系

很多人容易误解：

```text
优先级高
≠
一定占用CPU高
```

优先级决定的是：

```text
多个进程竞争CPU时
谁先获得CPU
```

例如：

* 进程A：NI=-10
* 进程B：NI=10

如果：

```text
A不需要CPU
```

那么：

```text
B仍然可以使用100%CPU
```

因此优先级影响的是**调度顺序**，而不是 CPU 使用上限。

---

## 八、常用命令汇总

| 操作      | 命令                                |
| ------- | --------------------------------- |
| 查看优先级   | `top`                             |
| 查看详细优先级 | `ps -eo pid,pri,ni,comm`          |
| 低优先级启动  | `nice -n 10 cmd`                  |
| 高优先级启动  | `sudo nice -n -10 cmd`            |
| 修改运行中进程 | `renice -10 -p PID`               |
| 查看调度策略  | `chrt -p PID`                     |
| 启动实时进程  | `sudo chrt -f 50 cmd`             |
| 查看实时优先级 | `ps -eo pid,class,rtprio,ni,comm` |

### 简单记忆

可以把 Linux 调度理解成银行排队：

* **Nice 值**：自己选择排队礼让程度。

  * `NI=19`：我不急，大家先来。
  * `NI=-20`：我有急事，请优先办理。

* **实时优先级（1~99）**：VIP 通道。

  * 普通队列再长，也会优先处理 VIP。
  * 设置不当可能导致其他进程长期得不到 CPU，因此生产环境应谨慎使用。
