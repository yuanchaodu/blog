---
title: Linux renice 命令
section: IT
category: Linux
---

#  Linux renice 命令

<img src="images/Linux.svg" width="300">

`renice` 是 Linux 中用于**修改正在运行进程优先级（nice 值）**的命令。

Linux 调度器会参考进程的 nice 值决定 CPU 分配倾向：

* nice 值范围：**-20 ～ 19**
* 数值越小（越负），优先级越高
* 数值越大，优先级越低
* 普通用户只能降低自己进程的优先级（增大 nice 值）
* root 用户可以设置任意 nice 值

---

## 一、命令语法

```bash
renice priority [[-p] pid ...]
renice priority -g pgrp ...
renice priority -u user ...
```

其中：

* `priority`：新的 nice 值
* `pid`：进程 ID
* `pgrp`：进程组
* `user`：用户

---

## 二、查看进程优先级

### 方法1：ps

```bash
ps -eo pid,user,ni,comm
```

输出示例：

```text
PID USER     NI COMMAND
1234 root     0 nginx
2345 oracle   5 java
3456 mysql   10 mysqld
```

其中：

* NI = Nice 值

---

### 方法2：top

```bash
top
```

显示：

```text
PID USER      PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
1234 root     20 0
```

字段说明：

* PR：实际调度优先级
* NI：nice 值

---

## 三、修改单个进程优先级

假设进程 PID 为 1234。

### 降低优先级

```bash
renice 10 -p 1234
```

输出：

```text
1234 (process ID) old priority 0, new priority 10
```

表示：

```text
0 → 10
```

CPU 资源获取机会减少。

---

### 提高优先级

需要 root 权限：

```bash
sudo renice -5 -p 1234
```

结果：

```text
0 → -5
```

CPU 调度优先级提高。

---

## 四、同时修改多个进程

```bash
sudo renice 5 -p 1234 2345 3456
```

---

## 五、按用户修改

将某个用户的所有进程调整为 nice=10：

```bash
sudo renice 10 -u oracle
```

例如：

```bash
sudo renice 10 -u tomcat
```

---

## 六、按进程组修改

```bash
sudo renice 5 -g 1001
```

其中：

```text
1001 = 进程组ID(PGID)
```

查看 PGID：

```bash
ps -efj
```

或：

```bash
ps -o pid,pgid,cmd
```

---

## 七、与 nice 命令的区别

| 命令     | 用途         |
| ------ | ---------- |
| nice   | 启动进程时指定优先级 |
| renice | 修改已运行进程优先级 |

### nice 示例

启动时设置：

```bash
nice -n 10 ./app
```

### renice 示例

运行后调整：

```bash
renice 10 -p 1234
```

---

## 八、实际运维场景

### 场景1：数据库占用 CPU 过高

发现：

```bash
top
```

```text
mysqld CPU 90%
```

降低优先级：

```bash
sudo renice 10 -p $(pidof mysqld)
```

---

### 场景2：批处理任务不影响生产系统

启动：

```bash
nice -n 15 python batch.py
```

或者运行后：

```bash
renice 15 -p PID
```

---

### 场景3：关键业务进程提高优先级

例如生产控制程序：

```bash
sudo renice -10 -p 1234
```

让系统优先调度该进程。

---

## 九、注意事项

### 1. renice 只影响 CPU 调度

不会影响：

* 内存分配
* 磁盘 I/O
* 网络带宽

如果需要控制 I/O 优先级，可使用：

```bash
ionice
```

例如：

```bash
sudo ionice -c2 -n7 -p 1234
```

---

### 2. nice 值不是实时优先级

Linux 实际调度时使用：

```text
实时优先级(PR)
+
nice值(NI)
+
调度策略
```

查看：

```bash
ps -eo pid,pri,ni,cmd
```

---

### 3. 实时任务不受 renice 影响

如果进程采用：

```text
SCHED_FIFO
SCHED_RR
```

实时调度策略，则需要使用：

```bash
chrt
```

查看：

```bash
chrt -p PID
```

---

## 十、常用命令速查

```bash
# 查看进程 nice 值
ps -eo pid,ni,cmd

# 修改进程优先级
renice 10 -p 1234

# 提高优先级（root）
sudo renice -5 -p 1234

# 修改多个进程
sudo renice 5 -p 1234 2345

# 修改某用户所有进程
sudo renice 10 -u oracle

# 查看实时优先级
ps -eo pid,pri,ni,cmd

# 查看调度策略
chrt -p 1234
```

可以把 `renice` 理解为：**运行中的“CPU 调度权重调节器”**，用于在不中断进程的情况下，动态调整其获得 CPU 时间片的机会。