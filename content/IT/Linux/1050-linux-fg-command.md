---
title: Linux fg 命令
section: IT
category: Linux
---

#  Linux fg 命令

<img src="images/Linux.svg" width="300">

`fg`（foreground）是 Linux Shell 中的一个**作业控制命令**，用于将后台运行或挂起的任务切换到前台运行。

## 基本语法

```bash
fg [%作业号]
```

例如：

```bash
fg %1
```

表示将编号为 1 的后台任务调回前台。

---

## 常见使用场景

### 1. 将当前程序挂起

假设运行一个程序：

```bash
vi test.txt
```

按下：

```text
Ctrl + Z
```

程序会被挂起（暂停），终端显示类似：

```bash
[1]+  Stopped                 vi test.txt
```

此时程序并没有退出，只是暂停了。

---

### 2. 查看作业列表

使用：

```bash
jobs
```

输出：

```bash
[1]+  Stopped                 vi test.txt
[2]-  Running                 ping 8.8.8.8 &
```

其中：

* `[1]`、`[2]` 为作业号（Job ID）
* `Stopped` 表示暂停
* `Running` 表示运行中

---

### 3. 使用 fg 恢复到前台

恢复作业 1：

```bash
fg %1
```

输出：

```bash
vi test.txt
```

编辑器重新回到前台。

---

## fg 与 bg 的区别

| 命令     | 作用            |
| ------ | ------------- |
| fg     | 将任务调到前台运行     |
| bg     | 将暂停任务放到后台继续运行 |
| jobs   | 查看当前作业        |
| Ctrl+Z | 挂起当前前台任务      |
| &      | 启动时直接放到后台运行   |

示例：

```bash
ping 8.8.8.8
```

按：

```text
Ctrl + Z
```

然后：

```bash
bg
```

输出：

```bash
[1]+ ping 8.8.8.8 &
```

任务继续在后台运行。

如果想重新切回前台：

```bash
fg %1
```

---

## fg 不带参数

如果只有一个后台任务：

```bash
fg
```

默认恢复最近一个后台任务。

例如：

```bash
jobs
```

```bash
[1]- Running  ping 8.8.8.8 &
[2]+ Stopped  vi test.txt
```

执行：

```bash
fg
```

会恢复带 `+` 标记的当前作业：

```bash
vi test.txt
```

---

## 实际运维中常见操作

### 场景1：误按 Ctrl+Z

```bash
top
```

误按：

```text
Ctrl + Z
```

恢复：

```bash
fg
```

---

### 场景2：后台运行后需要交互

```bash
python app.py &
```

查看：

```bash
jobs
```

```bash
[1]+ Running python app.py &
```

切回前台：

```bash
fg %1
```

---

### 场景3：编译任务后台执行

```bash
make
```

发现时间太长：

```text
Ctrl + Z
```

然后：

```bash
bg
```

让其后台继续编译。

之后查看：

```bash
jobs
```

需要观察输出时：

```bash
fg
```

---

## 注意事项

### 1. fg 只能操作当前 Shell 的作业

例如：

```bash
ssh server
```

在远程服务器启动的程序，如果关闭 SSH，会失去作业控制信息，`fg` 无法恢复。

长期运行任务建议使用：

* tmux
* GNU Screen
* `nohup`

例如：

```bash
nohup python app.py > app.log 2>&1 &
```

---

### 2. fg 不是进程管理命令

它管理的是 **Job（作业）**，不是 PID。

例如：

```bash
fg 1234
```

错误：

```bash
fg: 1234: no such job
```

因为 `1234` 被当成作业号，而不是进程号。

---

## 一张图理解

```text
前台运行
    │
Ctrl+Z
    ▼
暂停(Stopped)
    │
 ┌──┴──┐
 │     │
bg     fg
 │     │
 ▼     ▼
后台运行   前台运行
```

最常用的组合命令是：

```bash
Ctrl + Z
jobs
bg
fg
```

这四个命令基本覆盖了 Linux Shell 的作业控制操作。