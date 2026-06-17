---
title: Linux ps 命令
section: IT
category: Linux
---

# Linux ps 命令

<img src="images/Linux.svg" width="300">

`ps` 是 Linux 中查看**进程状态**的命令。它像“任务管理器”的文字版，用来查看当前有哪些程序正在运行。

## 常用命令

```bash
ps
```

查看当前终端下的进程。

```bash
ps -ef
```

查看系统中所有进程，常用。

```bash
ps aux
```

也是查看所有进程，显示内容更偏资源占用情况。

```bash
ps -ef | grep nginx
```

查找包含 `nginx` 的进程。

```bash
ps aux --sort=-%mem | head
```

按内存占用从高到低查看前几名。

```bash
ps aux --sort=-%cpu | head
```

按 CPU 占用从高到低查看前几名。

## 常见字段含义

以 `ps -ef` 为例：

```bash
UID   PID   PPID  C STIME TTY      TIME CMD
```

| 字段    | 含义          |
| ----- | ----------- |
| UID   | 启动进程的用户     |
| PID   | 进程 ID       |
| PPID  | 父进程 ID      |
| C     | CPU 使用情况    |
| STIME | 进程启动时间      |
| TTY   | 关联的终端       |
| TIME  | 累计占用 CPU 时间 |
| CMD   | 启动命令        |

## 常用场景

查看某个服务是否运行：

```bash
ps -ef | grep mysql
```

找到进程后结束它：

```bash
kill PID
```

强制结束：

```bash
kill -9 PID
```

更推荐先用普通 `kill`，不行再用 `kill -9`。