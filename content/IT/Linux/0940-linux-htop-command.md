---
title: Linux htop 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnR9u
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/94'
---

#  Linux htop 命令

<img src="images/Linux.svg" width="300">

`htop` 是 Linux 下一个非常常用的系统监控工具，可以看作是 `top` 命令的增强版。它以更直观的界面展示 CPU、内存、进程等信息，并支持鼠标操作和快捷键管理进程。

## 一、安装 htop

### CentOS/RHEL

```bash
yum install htop
```

或者：

```bash
dnf install htop
```

### Ubuntu/Debian

```bash
apt install htop
```

### openSUSE

```bash
zypper install htop
```

### 查看版本

```bash
htop --version
```

---

## 二、启动 htop

直接执行：

```bash
htop
```

界面大致如下：

```text
CPU[||||||||||||||| 35.2%]
Mem[|||||||||||||   4.2G/16G]
Swp[|              0K/2G]

PID USER  PRI NI VIRT RES SHR S CPU% MEM% TIME+ COMMAND
123 root   20  0 100M 20M 10M S 2.3  0.1 00:01 nginx
...
```

---

## 三、界面说明

### 顶部区域

显示系统整体资源情况：

* CPU使用率
* CPU核心状态
* 内存（Memory）使用情况
* Swap使用情况
* 系统运行时间（Uptime）
* 负载（Load Average）
* 进程数量

例如：

```text
Load average: 0.25 0.35 0.42
```

表示：

* 最近1分钟负载：0.25
* 最近5分钟负载：0.35
* 最近15分钟负载：0.42

---

### 进程列表区域

常见列说明：

| 列名      | 说明       |
| ------- | -------- |
| PID     | 进程ID     |
| USER    | 所属用户     |
| PRI     | 优先级      |
| NI      | Nice值    |
| VIRT    | 虚拟内存     |
| RES     | 实际占用物理内存 |
| SHR     | 共享内存     |
| S       | 进程状态     |
| CPU%    | CPU占用率   |
| MEM%    | 内存占用率    |
| TIME+   | 累计CPU时间  |
| COMMAND | 启动命令     |

---

### 进程状态

| 状态 | 含义           |
| -- | ------------ |
| R  | 运行中（Running） |
| S  | 休眠（Sleeping） |
| D  | 不可中断等待       |
| T  | 停止           |
| Z  | 僵尸进程（Zombie） |

---

## 四、常用快捷键

### 进程管理

| 快捷键   | 功能         |
| ----- | ---------- |
| F9    | 结束进程（Kill） |
| F7    | 提高优先级      |
| F8    | 降低优先级      |
| Space | 标记进程       |
| U     | 取消标记       |
| K     | 发送信号       |

---

### 查询和排序

| 快捷键 | 功能     |
| --- | ------ |
| F3  | 搜索进程   |
| F4  | 过滤进程   |
| F5  | 树状显示   |
| F6  | 选择排序字段 |
| F10 | 退出     |

例如：

按：

```text
F6
```

选择：

```text
CPU%
```

即可按CPU占用率排序。

---

### 常用排序快捷键

| 快捷键 | 功能      |
| --- | ------- |
| P   | 按CPU排序  |
| M   | 按内存排序   |
| T   | 按运行时间排序 |
| N   | 按PID排序  |

---

## 五、常见运维场景

### 1. 查找CPU占用最高的进程

启动：

```bash
htop
```

按：

```text
P
```

排序。

即可快速找到：

```text
java
python
mysqld
nginx
```

等高CPU进程。

---

### 2. 查找内存占用最高的进程

按：

```text
M
```

排序。

查看：

```text
RES
MEM%
```

列。

---

### 3. 树状查看父子进程

按：

```text
F5
```

显示：

```text
systemd
 ├─sshd
 │   └─bash
 │      └─python
 └─nginx
     ├─worker
     └─worker
```

非常适合分析服务启动链路。

---

### 4. 结束异常进程

选中进程后：

```text
F9
```

选择：

```text
15 SIGTERM
```

正常结束。

若无法结束：

```text
9 SIGKILL
```

强制结束。

---

## 六、命令行参数

### 只显示指定用户进程

```bash
htop -u oracle
```

### 显示树状结构

```bash
htop -t
```

### 延迟刷新时间

```bash
htop -d 20
```

单位为：

```text
1/10秒
```

即：

```text
20 = 2秒
```

---

## 七、与 top 的区别

| 功能   | top | htop |
| ---- | --- | ---- |
| 彩色界面 | ✗   | ✓    |
| 鼠标支持 | ✗   | ✓    |
| 树状进程 | ✗   | ✓    |
| 滚动查看 | ✗   | ✓    |
| 搜索进程 | 较弱  | ✓    |
| 进程操作 | 较复杂 | ✓    |

因此在日常运维中，很多管理员更倾向于使用 `htop`。

---

## 八、企业环境中的实用组合

### 查看CPU占用最高进程

```bash
htop
```

按 `P` 排序。

### 配合查看网络连接

```bash
ss -antp
```

### 配合查看磁盘IO

安装并使用：

```bash
iotop
```

### 配合查看系统整体性能

```bash
vmstat 1
```

### 配合查看内存

```bash
free -h
```

对于生产服务器故障排查，一个常见流程是：

```text
htop
 ↓
发现CPU或内存异常
 ↓
定位进程PID
 ↓
ps -fp PID
 ↓
查看日志
 ↓
分析原因
```

这样通常能快速定位系统性能瓶颈。
