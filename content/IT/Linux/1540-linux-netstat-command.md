---
title: Linux netstat 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AntmO
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/154'
---

# Linux netstat 命令

<img src="images/Linux.svg" width="300">

`netstat` 是 Linux 中用于查看**网络连接、端口监听、路由表和网络统计信息**的命令。

可以把它理解为“网络状态查看器”。

## 一、常用命令

### 1. 查看所有网络连接

```bash
netstat -a
```

包括：

* 已建立的连接
* 正在监听的端口
* TCP、UDP 连接

---

### 2. 查看正在监听的端口

```bash
netstat -l
```

常用写法：

```bash
netstat -lnt
```

参数含义：

* `-l`：只显示监听端口
* `-n`：直接显示 IP 地址和端口号，不解析名称
* `-t`：只显示 TCP

查看 UDP 监听端口：

```bash
netstat -lnu
```

同时查看 TCP 和 UDP：

```bash
netstat -lntup
```

其中：

* `-u`：UDP
* `-p`：显示对应进程和 PID

通常需要管理员权限：

```bash
sudo netstat -lntup
```

---

### 3. 查看已建立的 TCP 连接

```bash
netstat -ant
```

只筛选已建立连接：

```bash
netstat -ant | grep ESTABLISHED
```

---

### 4. 查看某个端口是否被占用

例如查看 8080 端口：

```bash
netstat -lntp | grep 8080
```

可能看到：

```text
tcp  0  0 0.0.0.0:8080  0.0.0.0:*  LISTEN  1234/java
```

表示：

* `8080` 端口正在监听
* 进程 PID 是 `1234`
* 程序是 `java`

---

### 5. 查看路由表

```bash
netstat -r
```

使用数字方式显示：

```bash
netstat -rn
```

---

### 6. 查看网卡统计信息

```bash
netstat -i
```

可以查看：

* 接收数据包数量
* 发送数据包数量
* 错误包
* 丢包情况

---

### 7. 查看协议统计信息

```bash
netstat -s
```

只查看 TCP 统计：

```bash
netstat -st
```

只查看 UDP 统计：

```bash
netstat -su
```

## 二、常用参数

| 参数   | 含义            |
| ---- | ------------- |
| `-a` | 显示所有连接和监听端口   |
| `-l` | 只显示监听端口       |
| `-n` | 直接显示 IP 和端口号  |
| `-t` | 显示 TCP        |
| `-u` | 显示 UDP        |
| `-p` | 显示进程 PID 和程序名 |
| `-r` | 显示路由表         |
| `-i` | 显示网卡信息        |
| `-s` | 显示协议统计信息      |
| `-c` | 持续刷新显示        |

例如，每隔一段时间持续刷新：

```bash
netstat -antc
```

## 三、输出字段说明

执行：

```bash
netstat -ant
```

可能得到：

```text
Proto Recv-Q Send-Q Local Address   Foreign Address  State
tcp   0      0      192.168.1.10:22 192.168.1.20:51682 ESTABLISHED
```

字段含义：

| 字段                | 含义             |
| ----------------- | -------------- |
| `Proto`           | 网络协议，如 TCP、UDP |
| `Recv-Q`          | 等待程序读取的数据量     |
| `Send-Q`          | 等待发送的数据量       |
| `Local Address`   | 本机 IP 和端口      |
| `Foreign Address` | 远程 IP 和端口      |
| `State`           | TCP 连接状态       |

常见状态：

* `LISTEN`：正在监听
* `ESTABLISHED`：连接已经建立
* `TIME_WAIT`：连接已关闭，等待系统清理
* `CLOSE_WAIT`：对方已关闭连接，本地程序尚未关闭
* `SYN_SENT`：正在请求建立连接
* `SYN_RECV`：已收到连接请求

## 四、现在更推荐使用 `ss`

一些新版本 Linux 默认不再安装 `netstat`。`netstat` 属于 `net-tools` 软件包，现在通常推荐使用 `ss` 命令。

对应关系如下：

```bash
netstat -lntp
```

可替换为：

```bash
ss -lntp
```

查看所有 TCP 连接：

```bash
ss -ant
```

查看 8080 端口：

```bash
ss -lntp | grep 8080
```

`ss` 的速度通常更快，在连接数量很多的服务器上更明显。

## 五、安装 netstat

在 Ubuntu、Debian 中：

```bash
sudo apt install net-tools
```

在 CentOS、RHEL 中：

```bash
sudo yum install net-tools
```

在较新的系统中也可以使用：

```bash
sudo dnf install net-tools
```

日常排查端口时，最常用的是：

```bash
sudo netstat -lntup
```

或者：

```bash
sudo ss -lntup
```
