---
title: Linux pkill 命令
section: IT
category: Linux
---

#  Linux pkill 命令

<img src="images/Linux.svg" width="300">

`pkill` 是 Linux 中根据**进程名称、用户、条件等信息查找并终止进程**的命令，可以理解为 **ps + grep + kill** 的组合简化版。

## 基本语法

```bash
pkill [选项] 进程名
```

例如：

```bash
pkill nginx
```

表示结束所有名称包含 `nginx` 的进程。

---

## 常用示例

### 1. 结束指定进程

```bash
pkill sshd
```

结束所有 sshd 进程。

---

### 2. 精确匹配进程名

默认情况下，`pkill` 使用模式匹配。

例如：

```bash
pkill java
```

可能会匹配：

```text
java
java_app
myjava
```

如果只想匹配完整进程名：

```bash
pkill -x java
```

---

### 3. 指定信号

默认发送的是：

```text
SIGTERM（15）
```

相当于：

```bash
pkill -TERM nginx
```

或：

```bash
pkill -15 nginx
```

强制结束：

```bash
pkill -9 nginx
```

即：

```bash
pkill -KILL nginx
```

---

### 4. 按用户结束进程

结束用户 `oracle` 的所有进程：

```bash
pkill -u oracle
```

结束 root 用户运行的 nginx：

```bash
pkill -u root nginx
```

---

### 5. 按终端结束进程

结束 tty1 上的所有进程：

```bash
pkill -t tty1
```

---

### 6. 使用正则表达式

结束所有 Python 程序：

```bash
pkill -f python
```

其中：

```bash
-f
```

表示匹配完整命令行。

例如：

```text
python app.py
python3 server.py
/usr/bin/python test.py
```

都会被匹配。

---

## 与 kill 的区别

### kill

需要知道 PID：

```bash
kill 12345
```

---

### pkill

根据名称查找 PID 并发送信号：

```bash
pkill nginx
```

相当于：

```bash
ps -ef | grep nginx
kill PID
```

---

## 与 killall 的区别

### killall

```bash
killall nginx
```

按进程名结束。

---

### pkill

```bash
pkill nginx
```

功能更强，可以：

* 按用户匹配
* 按终端匹配
* 按完整命令匹配
* 支持正则表达式

因此在脚本中更常用。

---

## 查看将要结束哪些进程

建议先使用 `pgrep` 查看：

```bash
pgrep nginx
```

或者：

```bash
pgrep -af nginx
```

显示：

```text
1234 nginx: master process
1235 nginx: worker process
```

确认无误后再执行：

```bash
pkill nginx
```

这是比较安全的做法。

---

## 常见运维场景

### 重启 Java 服务

```bash
pkill -f myapp.jar
```

然后：

```bash
nohup java -jar myapp.jar &
```

---

### 结束所有 Python 脚本

```bash
pkill -f python
```

---

### 结束当前用户所有进程

```bash
pkill -u $(whoami)
```

注意：这会导致当前登录会话也被终止。

---

## 常用参数汇总

| 参数    | 说明             |
| ----- | -------------- |
| `-f`  | 匹配完整命令行        |
| `-x`  | 精确匹配进程名        |
| `-u`  | 按用户匹配          |
| `-t`  | 按终端匹配          |
| `-9`  | 发送 SIGKILL     |
| `-15` | 发送 SIGTERM（默认） |
| `-l`  | 列出匹配进程名称，不执行终止 |
| `-c`  | 统计匹配数量         |

例如：

```bash
pkill -l nginx
```

查看匹配的进程名称。

统计匹配数量：

```bash
pkill -c nginx
```

---

### 推荐实践

生产环境中不要直接使用：

```bash
pkill -9
```

建议按照以下顺序：

```bash
pgrep -af nginx      # 先确认
pkill nginx          # 优雅停止
sleep 5
pkill -9 nginx       # 必要时强制停止
```

这样可以避免进程来不及保存数据或释放资源。