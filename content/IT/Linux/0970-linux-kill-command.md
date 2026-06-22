---
title: Linux kill 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSNp
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/97'
---

#  Linux kill 命令

<img src="images/Linux.svg" width="300">

`kill` 是 Linux 中用于向进程发送信号（Signal）的命令。虽然名字叫“kill”，但它不一定是结束进程，也可以让进程重载配置、暂停、继续运行等。

## 一、基本语法

```bash
kill [信号] PID
```

例如：

```bash
kill 1234
```

向 PID 为 1234 的进程发送默认信号 `SIGTERM(15)`。

---

## 二、常用信号

| 信号      | 编号 | 作用              |
| ------- | -- | --------------- |
| SIGTERM | 15 | 默认信号，请求进程正常退出   |
| SIGKILL | 9  | 强制终止进程          |
| SIGHUP  | 1  | 重新加载配置          |
| SIGINT  | 2  | 中断进程（类似 Ctrl+C） |
| SIGSTOP | 19 | 暂停进程            |
| SIGCONT | 18 | 恢复运行            |

查看系统支持的所有信号：

```bash
kill -l
```

输出类似：

```bash
HUP INT QUIT ILL TRAP ABRT BUS FPE
KILL USR1 SEGV USR2 PIPE ALRM TERM
...
```

---

## 三、常见用法

### 1. 正常结束进程

```bash
kill 1234
```

等同于：

```bash
kill -15 1234
```

或者：

```bash
kill -SIGTERM 1234
```

这种方式会给程序一个清理资源、保存数据的机会。

---

### 2. 强制结束进程

```bash
kill -9 1234
```

或者：

```bash
kill -SIGKILL 1234
```

适用于：

* 程序卡死
* 无法响应 SIGTERM

注意：

* 进程无法捕获 SIGKILL
* 可能导致数据未保存

因此建议优先使用：

```bash
kill PID
```

无效后再使用：

```bash
kill -9 PID
```

---

### 3. 结束多个进程

```bash
kill 1234 2345 3456
```

一次结束多个 PID。

---

### 4. 重新加载服务配置

例如 Nginx：

```bash
kill -HUP 1234
```

等同于：

```bash
kill -1 1234
```

很多守护进程收到 `SIGHUP` 后会重新读取配置文件。

---

### 5. 暂停和恢复进程

暂停：

```bash
kill -STOP 1234
```

恢复：

```bash
kill -CONT 1234
```

查看效果：

```bash
ps -ef
```

---

## 四、查找进程 PID

### 根据进程名查找

```bash
pgrep nginx
```

输出：

```bash
1234
```

---

### 使用 ps

```bash
ps -ef | grep nginx
```

输出：

```bash
root      1234     1  0 10:00 ? 00:00:01 nginx
```

PID 即第二列。

---

## 五、killall 与 pkill

### killall

按进程名结束：

```bash
killall nginx
```

结束所有 nginx 进程。

---

### pkill

根据名称匹配：

```bash
pkill nginx
```

或者：

```bash
pkill -9 java
```

结束所有 Java 进程。

---

## 六、实际运维场景

### 场景1：Java程序卡死

查看 PID：

```bash
ps -ef | grep java
```

先尝试：

```bash
kill PID
```

等待几秒后仍未退出：

```bash
kill -9 PID
```

---

### 场景2：重载 Nginx 配置

检查配置：

```bash
nginx -t
```

重载：

```bash
nginx -s reload
```

本质上相当于：

```bash
kill -HUP nginx主进程PID
```

---

### 场景3：结束占用端口的进程

查看 8080 端口：

```bash
lsof -i:8080
```

或者：

```bash
ss -lntp | grep 8080
```

得到 PID 后：

```bash
kill PID
```

---

## 七、kill 与 kill -9 的区别

| 项目      | kill        | kill -9    |
| ------- | ----------- | ---------- |
| 信号      | SIGTERM(15) | SIGKILL(9) |
| 是否可捕获   | 是           | 否          |
| 是否可清理资源 | 是           | 否          |
| 是否立即结束  | 不一定         | 是          |
| 推荐程度    | 优先使用        | 最后手段       |

运维实践中通常遵循：

```bash
kill PID
```

等待数秒无效后：

```bash
kill -9 PID
```

这样可以避免数据库、应用程序或文件系统出现数据损坏。
