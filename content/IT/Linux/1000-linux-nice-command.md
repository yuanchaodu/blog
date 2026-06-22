---
title: Linux nice 命令
section: IT
category: Linux
---

#  Linux nice 命令

<img src="images/Linux.svg" width="300">

`nice` 是 Linux 中用于**调整进程调度优先级**的命令。它不会直接决定进程占用多少 CPU，而是影响 Linux 调度器在多个进程竞争 CPU 时，优先安排哪个进程运行。

## 一、基本概念

Linux 中的优先级通常用 **Nice 值（NI）** 表示：

| Nice值 | 优先级 |
| ----- | --- |
| -20   | 最高  |
| 0     | 默认  |
| 19    | 最低  |

特点：

* Nice值越小（越负），优先级越高。
* Nice值越大（越正），优先级越低。
* 普通用户只能提高 Nice 值（降低优先级）。
* 只有 root 用户才能降低 Nice 值（提高优先级）。

可以理解为：

> Nice值表示“礼让程度”。值越大，越愿意把 CPU 让给别人。

---

## 二、基本语法

```bash
nice [OPTION] [COMMAND [ARG]...]
```

### 默认增加 10

例如：

```bash
nice tar -zcf backup.tar.gz /data
```

等同于：

```bash
nice -n 10 tar -zcf backup.tar.gz /data
```

启动的 `tar` 进程 Nice 值为：

```text
0 + 10 = 10
```

优先级低于普通进程。

---

## 三、指定 Nice 值

### 降低优先级

```bash
nice -n 15 python big_job.py
```

或者：

```bash
nice --adjustment=15 python big_job.py
```

结果：

```text
NI = 15
```

适合：

* 数据备份
* 日志分析
* 批量计算
* 数据导入

避免影响业务系统。

---

### 提高优先级（需要root）

```bash
sudo nice -n -10 python realtime.py
```

结果：

```text
NI = -10
```

适合：

* 实时数据采集
* 关键控制程序
* 高优先级业务

---

## 四、查看进程 Nice 值

### 使用 ps

```bash
ps -eo pid,user,ni,pri,cmd
```

输出示例：

```text
PID USER   NI PRI CMD
1234 root   0  19 nginx
5678 user  10  29 backup.sh
```

其中：

* NI：Nice值
* PRI：内核实际优先级

---

### 使用 top

```bash
top
```

显示：

```text
PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
1234 root 20  0 ...
5678 user 30 10 ...
```

其中：

* NI = Nice值
* PR = 调度优先级

---

## 五、修改正在运行的进程

`nice` 只能在启动时设置优先级。

对于已经运行的进程，需要使用 `renice`。

### 修改单个进程

```bash
renice 10 -p 1234
```

表示：

```text
PID 1234
NI -> 10
```

---

### 提高优先级

```bash
sudo renice -5 -p 1234
```

结果：

```text
NI = -5
```

---

### 按用户修改

```bash
sudo renice 15 -u oracle
```

把 oracle 用户所有进程的 Nice 值改为 15。

---

## 六、典型应用场景

### 场景1：数据库服务器备份

避免备份影响数据库运行：

```bash
nice -n 19 tar -zcf backup.tar.gz /data
```

---

### 场景2：大文件压缩

```bash
nice -n 15 gzip huge.log
```

后台慢慢执行，不抢占业务 CPU。

---

### 场景3：数据分析任务

```bash
nice -n 10 python analysis.py
```

生产环境常见做法。

---

### 场景4：关键业务进程

```bash
sudo nice -n -5 java -jar app.jar
```

提高关键服务响应速度。

---

## 七、与 CPU 使用率的关系

很多人误以为：

```bash
nice -n 19 command
```

会限制 CPU 使用率。

实际上：

❌ 不会。

例如系统空闲时：

```bash
nice -n 19 yes > /dev/null
```

仍然可能占满一个 CPU 核心：

```text
CPU = 100%
```

因为没有其他进程与其竞争。

`nice` 只在**CPU竞争发生时**起作用。

---

## 八、与 cgroups 的区别

| 功能        | nice | cgroups |
| --------- | ---- | ------- |
| 调整调度优先级   | √    | √       |
| 限制CPU占用比例 | ×    | √       |
| 限制内存      | ×    | √       |
| 限制IO      | ×    | √       |
| 精确资源控制    | ×    | √       |

例如：

```bash
systemd-run --scope -p CPUQuota=20% command
```

是真正限制 CPU 使用率。

---

## 九、实际生产环境建议

在服务器上运行耗时任务时，推荐：

```bash
nice -n 15 command
```

如果任务还有大量磁盘读写，再配合：

```bash
ionice -c3 command
```

或者：

```bash
ionice -c3 nice -n 15 command
```

这样既降低 CPU 优先级，又降低磁盘 I/O 优先级，对生产业务影响最小。

### 总结

* `nice`：启动进程时设置 CPU 调度优先级。
* Nice 范围：**-20 ～ 19**。
* 值越小，优先级越高。
* 值越大，优先级越低。
* 已运行进程使用 `renice` 修改。
* `nice` 不限制 CPU 使用率，只影响 CPU 竞争时的调度顺序。
* 生产环境中执行备份、压缩、数据分析等后台任务时，经常使用：

```bash
nice -n 15 command
```

配合 `ionice` 使用效果更好。