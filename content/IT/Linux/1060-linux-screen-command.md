---
title: Linux screen 命令
section: IT
category: Linux
---

#  Linux screen 命令

<img src="images/Linux.svg" width="300">

`screen` 是 Linux 下非常经典的 **终端复用工具**，可以让程序在后台持续运行，即使 SSH 断开也不会中断。

---

# 1. 安装 screen

Ubuntu / Debian：

```bash
sudo apt install screen
```

CentOS / RHEL：

```bash
sudo yum install screen
```

查看版本：

```bash
screen -v
```

---

# 2. 创建一个 screen 会话

```bash
screen
```

或者指定名字：

```bash
screen -S mysession
```

这样就进入了一个新的终端会话。

---

# 3. 临时离开（后台运行）

快捷键：

```text
Ctrl + A 然后按 D
```

会显示：

```text
[detached from 1234.mysession]
```

程序仍然在后台运行。

---

# 4. 查看所有 screen 会话

```bash
screen -ls
```

示例：

```text
There is a screen on:
    1234.mysession   (Detached)
```

---

# 5. 重新进入会话

```bash
screen -r mysession
```

或者：

```bash
screen -r 1234
```

---

# 6. 强制接管会话

如果提示：

```text
There is a screen on:
    1234.mysession (Attached)
```

使用：

```bash
screen -d -r mysession
```

意思：

* `-d`：踢掉旧连接
* `-r`：重新连接

---

# 7. 关闭 screen 会话

在 screen 内执行：

```bash
exit
```

或者：

```bash
Ctrl + D
```

---

# 8. 常用快捷键

先按：

```text
Ctrl + A
```

再按：

| 快捷键 | 功能     |
| --- | ------ |
| `c` | 新建窗口   |
| `n` | 下一个窗口  |
| `p` | 上一个窗口  |
| `"` | 窗口列表   |
| `d` | 后台分离   |
| `k` | 关闭当前窗口 |
| `A` | 重命名窗口  |

---

# 9. 实际常见用途

## 后台运行 Python

```bash
screen -S train
python train.py
```

断开：

```text
Ctrl+A D
```

重新连接：

```bash
screen -r train
```

---

## 后台运行下载任务

```bash
screen -S download
wget xxx.zip
```

---

## 长时间运行服务

```bash
screen -S server
./start.sh
```

---

# 10. screen 与 tmux 对比

现在很多人更喜欢用 tmux：

| screen | tmux  |
| ------ | ----- |
| 老牌稳定   | 功能更强  |
| 系统自带较多 | 配置现代化 |
| 简单易用   | 分屏强大  |

如果你刚开始学习，`screen` 已经足够用了。

---

# 11. 一条龙示例

```bash
# 创建会话
screen -S work

# 运行任务
python app.py

# 挂后台
Ctrl+A D

# 查看
screen -ls

# 恢复
screen -r work
```