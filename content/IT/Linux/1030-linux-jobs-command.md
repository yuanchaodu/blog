---
title: Linux jobs 命令
section: IT
category: Linux
---

#  Linux jobs 命令

<img src="images/Linux.svg" width="300">

`jobs` 是 Linux 中用于查看当前 Shell 及其子进程后台任务状态的命令，常用于作业控制（Job Control）。

## 基本语法

```bash
jobs [选项]
```

## 常见用法

### 1. 查看后台任务

例如启动一个后台任务：

```bash
sleep 300 &
```

输出：

```bash
[1] 12345
```

其中：

* `1`：作业号（Job ID）
* `12345`：进程 PID

查看当前后台任务：

```bash
jobs
```

输出：

```bash
[1]+ Running    sleep 300 &
```

---

### 2. 查看所有作业详细信息

```bash
jobs -l
```

输出：

```bash
[1]+ 12345 Running    sleep 300 &
```

显示 PID 信息。

---

### 3. 只显示 PID

```bash
jobs -p
```

输出：

```bash
12345
```

---

### 4. 查看运行中的任务

```bash
jobs -r
```

输出：

```bash
[1]+ Running    sleep 300 &
```

---

### 5. 查看已停止的任务

例如：

```bash
ping www.baidu.com
```

按：

```text
Ctrl + Z
```

任务暂停：

```bash
[1]+ Stopped    ping www.baidu.com
```

查看停止任务：

```bash
jobs -s
```

输出：

```bash
[1]+ Stopped    ping www.baidu.com
```

---

## 配合作业控制命令

### fg

将后台任务切换到前台：

```bash
fg %1
```

其中 `%1` 表示作业号 1。

---

### bg

让暂停任务在后台继续运行：

```bash
bg %1
```

输出：

```bash
[1]+ ping www.baidu.com &
```

---

### kill

结束指定作业：

```bash
kill %1
```

或者：

```bash
kill 12345
```

---

## 作业状态说明

| 状态         | 说明          |
| ---------- | ----------- |
| Running    | 正在运行        |
| Stopped    | 已暂停（Ctrl+Z） |
| Done       | 已完成         |
| Terminated | 已终止         |

例如：

```bash
jobs
```

输出：

```bash
[1]- Running    sleep 300 &
[2]+ Stopped    vim test.txt
```

含义：

* 作业1正在后台运行
* 作业2被暂停
* `+` 表示默认当前作业
* `-` 表示前一个作业

---

## jobs 与 ps 的区别

| 命令     | 查看范围         |
| ------ | ------------ |
| jobs   | 当前 Shell 的作业 |
| ps     | 系统中的进程       |
| top    | 系统实时进程       |
| pstree | 进程树          |

例如：

```bash
sleep 300 &
```

使用：

```bash
jobs
```

能看到该后台任务。

而：

```bash
ps -ef | grep sleep
```

则能看到系统中的 `sleep` 进程。

---

## 注意事项

`jobs` 是 Shell 内建命令（bash、zsh 等），不是独立可执行文件。

验证：

```bash
type jobs
```

输出：

```bash
jobs is a shell builtin
```

因此：

```bash
ssh 远程主机
```

或打开新的终端窗口后，原终端中的 `jobs` 信息是看不到的，因为作业信息只属于当前 Shell 会话。

## 实际运维场景

后台执行长时间任务：

```bash
tar -czf backup.tar.gz /data &
```

查看状态：

```bash
jobs -l
```

恢复到前台观察：

```bash
fg %1
```

如果希望退出终端后任务继续运行，建议使用：

```bash
nohup tar -czf backup.tar.gz /data > backup.log 2>&1 &
```

或者使用：

```bash
screen
```

或

```bash
tmux
```

这样即使 SSH 断开，任务仍会继续执行。