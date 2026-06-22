---
title: Linux nohup 命令
section: IT
category: Linux
---

#  Linux nohup 命令

<img src="images/Linux.svg" width="300">

`nohup`（No Hang Up）是 Linux 中用于**让程序在用户退出终端后继续运行**的命令。它通常与后台运行符号 `&` 配合使用。

## 一、基本语法

```bash
nohup 命令 &
```

例如：

```bash
nohup python app.py &
```

表示：

* 启动 `app.py`
* 放入后台运行
* 即使关闭 SSH 或终端，程序仍然继续执行

---

## 二、工作原理

正常情况下：

```text
终端关闭
    ↓
发送 SIGHUP 信号
    ↓
进程退出
```

使用 `nohup` 后：

```text
终端关闭
    ↓
SIGHUP 被忽略
    ↓
进程继续运行
```

---

## 三、输出日志

默认情况下：

```bash
nohup python app.py &
```

标准输出和错误输出会写入当前目录下的：

```text
nohup.out
```

查看日志：

```bash
tail -f nohup.out
```

---

## 四、指定日志文件

实际工作中更常见：

```bash
nohup python app.py > app.log 2>&1 &
```

含义：

| 部分        | 说明           |
| --------- | ------------ |
| `>`       | 重定向标准输出      |
| `app.log` | 日志文件         |
| `2>&1`    | 错误输出重定向到标准输出 |
| `&`       | 后台运行         |

所有日志都会写入：

```text
app.log
```

查看：

```bash
tail -f app.log
```

---

## 五、查看运行进程

### 方法1：ps

```bash
ps -ef | grep app.py
```

示例：

```bash
ps -ef | grep java
```

---

### 方法2：pgrep

```bash
pgrep -af app.py
```

结果更简洁：

```text
12345 python app.py
```

---

## 六、停止进程

先查看 PID：

```bash
ps -ef | grep app.py
```

假设 PID 为：

```text
12345
```

停止：

```bash
kill 12345
```

强制停止：

```bash
kill -9 12345
```

---

## 七、获取后台任务

查看当前 Shell 启动的后台任务：

```bash
jobs -l
```

输出：

```text
[1]+ 12345 Running nohup python app.py &
```

---

## 八、常见用法

### 1. Java 服务

```bash
nohup java -jar app.jar > app.log 2>&1 &
```

---

### 2. Shell脚本

```bash
nohup sh backup.sh > backup.log 2>&1 &
```

---

### 3. Python程序

```bash
nohup python3 server.py > server.log 2>&1 &
```

---

### 4. Spring Boot项目

```bash
nohup java -Xms2g -Xmx4g -jar demo.jar > demo.log 2>&1 &
```

---

## 九、查看启动是否成功

启动：

```bash
nohup java -jar app.jar > app.log 2>&1 &
```

返回：

```text
[1] 12345
```

其中：

* `[1]`：任务编号
* `12345`：进程PID

验证：

```bash
ps -p 12345
```

或者：

```bash
netstat -tunlp | grep 端口号
```

现代 Linux 也可使用：

```bash
ss -tunlp | grep 端口号
```

---

## 十、nohup 与 & 的区别

### 仅使用 &

```bash
python app.py &
```

特点：

* 后台运行
* 关闭终端后可能退出

---

### 使用 nohup + &

```bash
nohup python app.py &
```

特点：

* 后台运行
* 关闭终端后仍运行

---

## 十一、生产环境更推荐的方式

对于长期运行的服务（Java、Python、Nginx、自研应用等），建议使用：

* Linux Systemd
* Supervisor
* Docker
* Kubernetes

例如在采用 `systemd` 的服务器上：

```bash
systemctl start myapp
systemctl stop myapp
systemctl restart myapp
```

相比 `nohup`：

* 自动拉起
* 开机自启
* 日志统一管理
* 运行状态可监控

因此，`nohup` 更适合临时任务、测试程序或简单后台作业；生产环境长期服务通常优先使用 `systemd` 等进程管理工具。