---
title: Linux bg 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSnj
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/104'
---

#  Linux bg 命令

<img src="images/Linux.svg" width="300">

`bg`（background）是 Linux Shell 中用于**将已暂停的作业放到后台继续运行**的命令。

## 基本语法

```bash
bg
```

或

```bash
bg %作业号
```

其中：

* `%1` 表示作业1
* `%2` 表示作业2
* 以此类推

---

## 常见使用场景

### 1. 暂停当前程序后放入后台

假设正在运行一个耗时程序：

```bash
find / -name "*.log"
```

此时想暂时释放终端，可以按：

```bash
Ctrl + Z
```

系统显示：

```bash
[1]+ Stopped find / -name "*.log"
```

表示作业1已暂停。

然后执行：

```bash
bg
```

输出：

```bash
[1]+ find / -name "*.log &
```

此时程序会在后台继续执行。

---

### 2. 查看后台作业

使用：

```bash
jobs
```

例如：

```bash
$ jobs

[1]- Running    find / -name "*.log" &
[2]+ Running    ping 8.8.8.8 &
```

---

### 3. 指定某个作业转入后台

如果有多个暂停作业：

```bash
jobs
```

显示：

```bash
[1]- Stopped    vi test.txt
[2]+ Stopped    top
```

让作业2继续在后台运行：

```bash
bg %2
```

---

## bg 与 & 的区别

### 方法1：启动时直接后台运行

```bash
python test.py &
```

程序从开始就在后台运行。

### 方法2：运行后再转后台

```bash
python test.py
```

按：

```bash
Ctrl + Z
```

再执行：

```bash
bg
```

程序继续在后台运行。

---

## 配合 fg 使用

### fg（foreground）

将后台任务调回前台：

```bash
fg
```

或：

```bash
fg %1
```

例如：

```bash
jobs

[1]+ Running ping 8.8.8.8 &
```

恢复到前台：

```bash
fg %1
```

此时终端重新接管该程序。

---

## 注意事项

### 1. 不是所有程序都适合 bg

例如：

```bash
vi
nano
top
htop
```

这些交互式程序依赖终端输入输出，即使使用 `bg`，也可能立即暂停或无法正常工作。

---

### 2. 关闭终端后任务可能终止

普通的：

```bash
bg
```

只是放到后台运行。

如果退出 SSH 或关闭终端，进程通常会收到 `SIGHUP` 信号而退出。

要长期运行可使用：

```bash
nohup command &
```

或者：

```bash
screen
```

或：

```bash
tmux
```

---

## 典型工作流程

```bash
# 前台运行
./backup.sh

# 按 Ctrl+Z 暂停
^Z

# 后台继续执行
bg

# 查看任务
jobs

# 调回前台
fg
```

流程图：

```text
前台运行
    │
Ctrl+Z
    ↓
暂停(Stopped)
    │
   bg
    ↓
后台运行(Running)
    │
   fg
    ↓
前台运行
```

简单来说：

* `Ctrl+Z`：暂停当前任务
* `bg`：让暂停的任务在后台继续运行
* `jobs`：查看当前作业
* `fg`：把后台任务切回前台
* `&`：启动时直接放到后台运行
* `nohup`：终端关闭后仍继续运行程序
