---
title: Linux 进程基础
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnLhS
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/91'
---

# Linux 进程基础

<img src="images/Linux.svg" width="300">

# Linux 进程基础

进程（Process）是 Linux 操作系统中最核心的概念之一。简单来说：

> **程序是静态的文件，进程是程序运行时的实例。**

例如，磁盘上的 `nginx` 程序只是一个文件，当执行它后，系统会创建一个或多个进程来运行它。

---

## 一、什么是进程

### 通俗解释

如果把程序比作一本菜谱：

* 菜谱（程序）是静态的
* 按照菜谱做菜的过程（进程）是动态的

同一本菜谱可以同时被很多厨师使用，因此：

* 一个程序可以对应多个进程
* 每个进程都有自己的运行状态

例如：

```bash
python app.py
```

每执行一次，就会产生一个新的 Python 进程。

---

## 二、进程包含哪些内容

一个 Linux 进程通常包括：

| 组成部分      | 说明           |
| --------- | ------------ |
| 代码段（Text） | 程序执行代码       |
| 数据段（Data） | 全局变量、静态变量    |
| 堆（Heap）   | 动态申请内存       |
| 栈（Stack）  | 函数调用和局部变量    |
| 打开的文件     | 文件、Socket等资源 |
| 环境变量      | PATH、HOME等   |
| 进程状态      | 运行、睡眠等       |
| PID       | 进程唯一标识       |

---

## 三、进程的主要属性

### 1. PID（Process ID）

每个进程都有唯一的进程号。

查看当前 Shell 的 PID：

```bash
echo $$
```

查看所有进程：

```bash
ps -ef
```

示例：

```text
UID       PID   PPID  CMD
root     1024      1  nginx
```

其中：

* PID=1024：当前进程编号
* PPID=1：父进程编号

---

### 2. 父进程与子进程

Linux 中所有进程都由其他进程创建。

例如：

```text
systemd
 └── bash
      └── python
```

这里：

* systemd 是 bash 的父进程
* bash 是 python 的父进程

查看进程树：

```bash
pstree
```

或者：

```bash
pstree -p
```

显示 PID：

```text
systemd(1)
 └─bash(2010)
     └─python(3010)
```

---

### 3. 用户身份

每个进程都属于某个用户。

查看：

```bash
ps -ef
```

第一列：

```text
root
nginx
oracle
```

表示进程所属用户。

---

## 四、进程状态

Linux 中常见进程状态：

| 状态 | 含义             |
| -- | -------------- |
| R  | Running，运行     |
| S  | Sleeping，可中断睡眠 |
| D  | 不可中断睡眠         |
| T  | 停止             |
| Z  | 僵尸进程           |
| X  | 已终止            |

查看：

```bash
ps aux
```

示例：

```text
USER PID %CPU %MEM VSZ RSS TTY STAT COMMAND
root 1234 0.0 0.1 ... S nginx
```

其中：

```text
STAT=S
```

表示睡眠状态。

---

## 五、进程生命周期

一个进程通常经历：

```text
创建
 ↓
就绪
 ↓
运行
 ↓
等待
 ↓
运行
 ↓
结束
```

更完整地表示：

```text
          +---------+
          | Ready   |
          +---------+
               |
               v
          +---------+
          | Running |
          +---------+
            /     \
           /       \
          v         v
   +---------+   +---------+
   | Waiting |   | Exit    |
   +---------+   +---------+
```

---

## 六、进程创建机制

Linux 中创建进程主要依靠：

### fork()

复制当前进程。

```c
pid_t pid = fork();
```

执行后：

```text
父进程
   │
 fork()
   │
   ├──父进程继续执行
   └──子进程开始执行
```

特点：

* 子进程拥有新的 PID
* 继承父进程资源
* 使用写时复制（Copy-On-Write）

---

### exec()

加载新的程序。

例如：

```c
execl("/bin/ls","ls","-l",NULL);
```

作用：

```text
原进程
   ↓
替换为新程序
```

PID 不变。

---

### fork + exec

Linux 中最常见模式：

```text
Shell
  │
fork
  │
子进程
  │
exec
  │
执行 ls
```

例如：

```bash
ls -l
```

Shell 实际上就是：

```text
bash
 └── fork
      └── exec(ls)
```

---

## 七、进程调度

CPU 同一时刻只能运行有限数量的进程。

Linux 调度器负责：

* 分配 CPU
* 切换进程
* 控制优先级

查看优先级：

```bash
ps -el
```

或：

```bash
top
```

常见参数：

| 项目   | 说明     |
| ---- | ------ |
| PRI  | 动态优先级  |
| NI   | Nice值  |
| %CPU | CPU占用率 |

---

### 调整优先级

启动时：

```bash
nice -n 10 python app.py
```

修改运行中的进程：

```bash
renice 10 -p 1234
```

---

## 八、常用进程管理命令

### ps

查看进程

```bash
ps -ef
```

查看指定进程：

```bash
ps -ef | grep nginx
```

---

### top

动态监控：

```bash
top
```

增强版：

```bash
htop
```

如果安装了的话。

---

### pgrep

查找 PID：

```bash
pgrep nginx
```

---

### pidof

查看程序 PID：

```bash
pidof nginx
```

---

### kill

结束进程：

```bash
kill 1234
```

强制结束：

```bash
kill -9 1234
```

---

### pkill

按名称结束：

```bash
pkill nginx
```

---

### killall

结束所有同名进程：

```bash
killall nginx
```

---

## 九、僵尸进程与孤儿进程

### 僵尸进程（Zombie）

状态：

```text
Z
```

产生原因：

```text
子进程退出
↓
父进程未回收
↓
形成僵尸进程
```

查看：

```bash
ps -el | grep Z
```

特点：

* 不占 CPU
* 几乎不占内存
* 占用 PID

---

### 孤儿进程（Orphan）

产生原因：

```text
父进程退出
↓
子进程仍运行
```

此时会被 PID 1 接管：

```text
systemd(1)
```

自动负责回收。

---

## 十、线程与进程区别

| 项目   | 进程  | 线程     |
| ---- | --- | ------ |
| 资源空间 | 独立  | 共享     |
| 地址空间 | 独立  | 共享     |
| 创建开销 | 大   | 小      |
| 切换开销 | 大   | 小      |
| 稳定性  | 高   | 较低     |
| 通信   | IPC | 直接共享内存 |

简单理解：

* **进程像一个独立公司**
* **线程像公司内部员工**

多个员工共享公司资源，但彼此分工合作。

---

## 十一、Linux 运维最常用的进程排查命令

### 查看 CPU 消耗最高进程

```bash
top
```

或：

```bash
ps aux --sort=-%cpu | head
```

---

### 查看内存消耗最高进程

```bash
ps aux --sort=-%mem | head
```

---

### 查看进程打开文件

```bash
lsof -p PID
```

---

### 查看进程网络连接

```bash
ss -pnt
```

或者：

```bash
netstat -antp
```

---

### 查看进程工作目录

```bash
pwdx PID
```

---

### 查看进程详细信息

```bash
cat /proc/PID/status
```

例如：

```bash
cat /proc/1234/status
```

---

## 十二、进程管理知识体系

可以按下面的路线理解 Linux 进程：

```text
进程基础
│
├── PID与PPID
├── 进程状态
├── fork/exec
├── 调度机制
├── 信号机制
├── 线程
├── IPC进程通信
├── 守护进程
├── systemd服务管理
├── /proc文件系统
└── 性能分析与故障排查
```

其中，实际运维和系统管理工作中最重要的内容是：

1. 进程状态（R、S、D、Z）
2. fork/exec 创建机制
3. 信号与 kill 命令
4. systemd 服务管理
5. `/proc` 文件系统
6. CPU、内存和 I/O 问题定位

掌握这几部分后，就能够分析大多数 Linux 应用服务（如 Nginx、MySQL、Redis、Java 应用等）的运行情况和故障问题。
