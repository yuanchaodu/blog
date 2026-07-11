---
title: Linux ss 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4Antk8
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/153'
---

# Linux ss 命令

<img src="images/Linux.svg" width="300">

`ss`（**Socket Statistics**）是 Linux 中用于查看网络连接和 Socket 信息的工具，属于 `iproute2` 套件。它是 `netstat` 的替代品，速度更快、功能更丰富，特别是在连接数量很多的服务器上优势明显。

---

# 一、ss 命令作用

`ss` 可以查看：

* TCP/UDP 连接
* Socket 状态
* 监听端口
* 进程占用端口
* 网络统计信息
* UNIX Socket
* SCTP、RAW Socket 等

例如：

```
客户端
    │
    ▼
Linux Socket
    │
    ▼
TCP/IP 协议栈
    │
    ▼
网络接口
```

`ss` 就是查看 Socket 当前状态的工具。

---

# 二、基本语法

```bash
ss [选项]
```

最常用：

```bash
ss -tulnp
```

很多运维工程师几乎每天都会使用。

---

# 三、常用参数

| 参数 | 含义               |
| -- | ---------------- |
| -a | 显示全部 Socket      |
| -l | 仅显示监听（Listening） |
| -t | TCP              |
| -u | UDP              |
| -x | Unix Socket      |
| -n | 数字显示，不解析主机名和端口   |
| -p | 显示进程             |
| -e | 扩展信息             |
| -m | Socket 内存        |
| -o | Timer 信息         |
| -s | 汇总统计             |
| -4 | IPv4             |
| -6 | IPv6             |

---

# 四、最常用命令

## 1）查看所有 TCP 连接

```bash
ss -t
```

输出示例：

```
State      Recv-Q Send-Q Local Address:Port Peer Address:Port
ESTAB      0      0      192.168.1.10:ssh  192.168.1.20:51234
```

表示：

* 当前 TCP 已建立连接（ESTAB）
* 本地 SSH 服务
* 对端客户端

---

## 2）查看所有 UDP

```bash
ss -u
```

---

## 3）查看所有监听端口（最常用）

```bash
ss -l
```

例如：

```
LISTEN
LISTEN
LISTEN
```

---

## 4）查看监听 TCP

```bash
ss -lt
```

---

## 5）查看监听 UDP

```bash
ss -lu
```

---

## 6）查看监听端口 + 进程

```bash
sudo ss -ltnp
```

例如：

```
State   Recv-Q Send-Q Local Address:Port

LISTEN  0      128    0.0.0.0:22

users:(("sshd",pid=678,fd=3))
```

解释：

```
22端口
    │
sshd
    │
PID=678
```

这说明：

SSH 服务正在监听 22 端口。

---

## 7）查看所有连接（最常用）

```bash
ss -tunap
```

参数含义：

```
-t TCP
-u UDP
-n 数字显示
-a 所有
-p 进程
```

输出会包括：

* LISTEN
* ESTAB
* TIME_WAIT
* CLOSE_WAIT

等所有状态。

---

# 五、查看指定端口

例如查看 80：

```bash
ss -ltn '( sport = :80 )'
```

查看 443：

```bash
ss -ltn '( sport = :443 )'
```

---

# 六、查看指定进程

例如 ssh：

```bash
ss -tp | grep ssh
```

或者：

```bash
ss -ltnp | grep sshd
```

---

# 七、查看连接状态

查看所有建立连接：

```bash
ss -tan state established
```

查看 TIME_WAIT：

```bash
ss -tan state time-wait
```

查看 CLOSE_WAIT：

```bash
ss -tan state close-wait
```

查看 LISTEN：

```bash
ss -tan state listening
```

也可以一次查看多个状态：

```bash
ss -tan state connected
```

---

# 八、查看统计信息

```bash
ss -s
```

例如：

```
Total: 950

TCP: 400

UDP: 30

RAW: 2

INET: 432

FRAG: 0
```

还会显示 TCP 各种状态统计：

```
TCP:
    established 32
    closed 300
    orphaned 1
    timewait 80
```

这是快速判断服务器连接情况的常用命令。

---

# 九、查看 Socket 内存

```bash
ss -m
```

输出包括：

```
skmem
rmem_alloc
wmem_alloc
```

可用于分析网络缓冲区使用情况。

---

# 十、查看 Timer 信息

```bash
ss -o
```

例如：

```
timer:(keepalive,119min,0)
```

表示：

* Keepalive 定时器
* 剩余时间

---

# 十一、过滤条件

查看来源 IP：

```bash
ss dst 192.168.1.100
```

查看目标 IP：

```bash
ss src 192.168.1.10
```

查看指定端口：

```bash
ss sport = :22
```

查看目标端口：

```bash
ss dport = :443
```

组合过滤：

```bash
ss -tn sport = :22
```

或：

```bash
ss -tn '( sport = :22 )'
```

---

# 十二、常见状态说明

| 状态         | 含义                 |
| ---------- | ------------------ |
| LISTEN     | 服务正在监听端口           |
| ESTAB      | 已建立连接（Established） |
| SYN-SENT   | 已发送 SYN，等待对方响应     |
| SYN-RECV   | 已收到 SYN，等待确认       |
| FIN-WAIT-1 | 主动关闭，等待对方确认        |
| FIN-WAIT-2 | 已收到确认，等待对方关闭       |
| CLOSE-WAIT | 收到对方关闭请求，等待本地应用关闭  |
| LAST-ACK   | 等待最后一个 ACK         |
| TIME-WAIT  | 主动关闭后等待，防止旧报文影响新连接 |
| CLOSED     | 连接已关闭              |

其中：

* **ESTAB**：正常业务连接。
* **LISTEN**：服务正常监听。
* **TIME-WAIT**：短连接较多时数量可能较大，一般属于正常现象。
* **CLOSE-WAIT**：持续增长通常意味着应用程序没有及时关闭 Socket，需要重点排查。

---

# 十三、运维常用示例

```bash
# 查看所有监听端口
ss -ltn

# 查看监听端口及对应进程
sudo ss -ltnp

# 查看所有 TCP 连接
ss -tan

# 查看已建立连接
ss -tan state established

# 查看 80 端口连接
ss -tn '( sport = :80 )'

# 查看连接数最多的 IP
ss -tn | awk 'NR>1 {print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head

# 查看 TCP 状态统计
ss -s

# 查看所有 UNIX Socket
ss -x
```

## 与 `netstat` 的对比

| 对比项  | `ss`                | `netstat`                      |
| ---- | ------------------- | ------------------------------ |
| 性能   | 更快，直接读取内核数据         | 较慢，连接数多时尤其明显                   |
| 默认安装 | 大多数现代 Linux 发行版默认提供 | 在部分发行版中已不默认安装（需安装 `net-tools`） |
| 功能   | 更丰富，支持更多过滤和统计       | 功能较基础                          |
| 推荐程度 | ★★★★★（推荐）           | ★★☆☆☆（兼容旧系统时使用）                |

对于现代 Linux 系统（如 RHEL 7+/8+/9+、CentOS 7+、Rocky Linux、AlmaLinux、Ubuntu、Debian 等），建议优先使用 `ss`，因为它性能更高、功能更完善，也是官方推荐的网络连接诊断工具。
