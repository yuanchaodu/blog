---
title: Linux tmux 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSvv
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/107'
---

#  Linux tmux 命令

<img src="images/Linux.svg" width="300">

`tmux`（Terminal Multiplexer）是 Linux 下非常常用的终端复用工具。它允许你在一个 SSH 会话中创建多个终端窗口，即使网络断开，程序也能继续运行。

## 一、安装 tmux

### CentOS / Rocky / RHEL

```bash
sudo yum install tmux
```

或：

```bash
sudo dnf install tmux
```

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install tmux
```

### 查看版本

```bash
tmux -V
```

---

## 二、会话（Session）管理

### 创建新会话

```bash
tmux
```

或指定名称：

```bash
tmux new -s mysession
```

### 查看所有会话

```bash
tmux ls
```

例如：

```text
mysession: 1 windows (created Mon Jun 22 10:00:00 2026)
```

### 进入已有会话

```bash
tmux attach -t mysession
```

简写：

```bash
tmux a -t mysession
```

### 退出会话（保持程序运行）

按：

```text
Ctrl+b d
```

此时会返回普通终端，但 tmux 中的程序继续运行。

### 关闭会话

进入会话后执行：

```bash
exit
```

或者：

```bash
tmux kill-session -t mysession
```

### 关闭所有会话

```bash
tmux kill-server
```

---

## 三、窗口（Window）管理

tmux 中一个 Session 可以包含多个 Window。

### 创建新窗口

快捷键：

```text
Ctrl+b c
```

### 查看窗口列表

```text
Ctrl+b w
```

### 切换窗口

下一窗口：

```text
Ctrl+b n
```

上一窗口：

```text
Ctrl+b p
```

指定窗口：

```text
Ctrl+b 0
Ctrl+b 1
Ctrl+b 2
```

### 重命名窗口

```text
Ctrl+b ,
```

### 关闭窗口

```bash
exit
```

或者：

```text
Ctrl+b &
```

---

## 四、窗格（Pane）管理

一个窗口可以分成多个 Pane（分屏）。

### 水平分屏

```text
Ctrl+b "
```

效果：

```text
+----------------+
|      Pane1     |
+----------------+
|      Pane2     |
+----------------+
```

### 垂直分屏

```text
Ctrl+b %
```

效果：

```text
+---------+------+
| Pane1   |Pane2 |
+---------+------+
```

### 切换窗格

```text
Ctrl+b 方向键
```

例如：

```text
Ctrl+b →
```

### 显示窗格编号

```text
Ctrl+b q
```

### 调整窗格大小

进入调整模式：

```text
Ctrl+b :
```

输入：

```bash
resize-pane -L 10
resize-pane -R 10
resize-pane -U 5
resize-pane -D 5
```

---

## 五、复制与滚动查看

### 进入复制模式

```text
Ctrl+b [
```

此时可以使用：

```text
↑ ↓ PgUp PgDn
```

浏览历史输出。

### 退出复制模式

```text
q
```

### 搜索内容

进入复制模式后：

```text
/
```

输入关键字搜索。

---

## 六、常用快捷键汇总

| 功能     | 快捷键        |
| ------ | ---------- |
| 创建窗口   | Ctrl+b c   |
| 查看窗口列表 | Ctrl+b w   |
| 下一窗口   | Ctrl+b n   |
| 上一窗口   | Ctrl+b p   |
| 重命名窗口  | Ctrl+b ,   |
| 水平分屏   | Ctrl+b "   |
| 垂直分屏   | Ctrl+b %   |
| 切换窗格   | Ctrl+b 方向键 |
| 显示窗格编号 | Ctrl+b q   |
| 分离会话   | Ctrl+b d   |
| 进入复制模式 | Ctrl+b [   |
| 显示帮助   | Ctrl+b ?   |

---

## 七、运维常用场景

### 后台运行脚本

```bash
tmux new -s backup
```

执行：

```bash
./backup.sh
```

按：

```text
Ctrl+b d
```

退出 SSH 后脚本仍然运行。

重新连接：

```bash
tmux attach -t backup
```

---

### 查看日志

一个窗口分成两个窗格：

左边：

```bash
tail -f /var/log/messages
```

右边：

```bash
top
```

同时观察日志和系统资源。

---

### 多服务器维护

创建多个窗口：

```text
Window1 -> 生产环境
Window2 -> 测试环境
Window3 -> 数据库
```

通过：

```text
Ctrl+b n
```

快速切换。

---

## 八、常用 tmux 命令速查

```bash
# 创建会话
tmux new -s mysession

# 查看会话
tmux ls

# 连接会话
tmux attach -t mysession

# 分离会话
Ctrl+b d

# 关闭会话
tmux kill-session -t mysession

# 关闭所有会话
tmux kill-server

# 新建窗口
Ctrl+b c

# 水平分屏
Ctrl+b "

# 垂直分屏
Ctrl+b %

# 查看帮助
Ctrl+b ?
```

如果你经常通过 SSH 维护 Linux 服务器，最值得掌握的其实只有 5 个操作：

```bash
tmux new -s work      # 创建会话
Ctrl+b c             # 新建窗口
Ctrl+b %             # 左右分屏
Ctrl+b "             # 上下分屏
Ctrl+b d             # 断开但不退出
tmux attach -t work  # 重新连接
```

掌握这几个命令，已经可以应对 90% 的服务器运维场景。
