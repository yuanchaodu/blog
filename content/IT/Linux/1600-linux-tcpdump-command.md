---
title: Linux tcpdump 抓包
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4Anuk0
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/160'
---

# Linux tcpdump 抓包

<img src="images/Linux.svg" width="300">

`tcpdump` 是 Linux 下最常用的命令行抓包工具，可以直接抓取网络接口上的数据包，功能类似 Wireshark，但更加轻量，非常适合服务器排查网络问题。

下面按照常用场景介绍。

---

# 一、安装 tcpdump

不同发行版安装方式不同。

## CentOS/RHEL

```bash
yum install tcpdump
```

或者

```bash
dnf install tcpdump
```

## Ubuntu/Debian

```bash
sudo apt install tcpdump
```

查看版本

```bash
tcpdump --version
```

---

# 二、查看网卡

先查看有哪些网卡。

```bash
tcpdump -D
```

例如输出

```text
1.eth0
2.lo
3.docker0
```

或者使用

```bash
ip addr
```

查看

```text
eth0
ens160
enp3s0
bond0
```

---

# 三、最简单抓包

抓取所有数据

```bash
sudo tcpdump
```

或者指定网卡

```bash
sudo tcpdump -i eth0
```

如果不知道网卡

```bash
sudo tcpdump -i any
```

表示监听所有接口。

---

# 四、不解析域名（推荐）

默认 tcpdump 会解析 IP、端口，影响速度。

建议加 `-n`

```bash
sudo tcpdump -i eth0 -n
```

如果连 MAC 地址也不解析

```bash
sudo tcpdump -i eth0 -nn
```

例如输出

```text
10.10.1.2.54321 > 10.10.1.100.80
```

---

# 五、抓指定主机

抓某个 IP

```bash
tcpdump -i eth0 host 192.168.1.100
```

抓源地址

```bash
tcpdump src host 192.168.1.100
```

抓目的地址

```bash
tcpdump dst host 192.168.1.100
```

---

# 六、抓指定端口

抓 80

```bash
tcpdump port 80
```

抓 TCP 443

```bash
tcpdump tcp port 443
```

抓 UDP 53

```bash
tcpdump udp port 53
```

多个端口

```bash
tcpdump 'port 80 or port 443'
```

---

# 七、抓指定协议

TCP

```bash
tcpdump tcp
```

UDP

```bash
tcpdump udp
```

ICMP

```bash
tcpdump icmp
```

ARP

```bash
tcpdump arp
```

---

# 八、组合条件

例如

抓来自某 IP 的 HTTP

```bash
tcpdump src host 10.0.0.1 and port 80
```

抓不是 SSH

```bash
tcpdump not port 22
```

抓两个 IP

```bash
tcpdump host 192.168.1.1 or host 192.168.1.2
```

抓 TCP 且不是 22

```bash
tcpdump tcp and not port 22
```

---

# 九、保存抓包文件

这是最常见的方式。

```bash
tcpdump -i eth0 -w test.pcap
```

指定数量

```bash
tcpdump -c 1000 -w test.pcap
```

抓 1000 个包停止。

---

# 十、读取 pcap

```bash
tcpdump -r test.pcap
```

详细输出

```bash
tcpdump -nn -r test.pcap
```

---

# 十一、查看数据内容

ASCII

```bash
tcpdump -A
```

Hex

```bash
tcpdump -X
```

Hex + ASCII

```bash
tcpdump -XX
```

例如

```bash
tcpdump -i eth0 -A port 80
```

可以直接看到 HTTP 请求。

---

# 十二、控制抓包长度

默认只抓部分数据。

抓完整包

```bash
tcpdump -s 0
```

例如

```bash
tcpdump -i eth0 -s 0 -w full.pcap
```

这是生产环境常用参数。

---

# 十三、限制抓包数量

```bash
tcpdump -c 100
```

表示抓 100 个包退出。

---

# 十四、按文件大小滚动

例如每 100MB 一个文件

```bash
tcpdump -i eth0 -C 100 -w capture.pcap
```

生成

```text
capture.pcap0
capture.pcap1
capture.pcap2
```

---

# 十五、按时间滚动

每 60 秒一个文件

```bash
tcpdump -G 60 -w "capture-%Y%m%d-%H%M%S.pcap"
```

例如

```text
capture-20250705-100000.pcap
capture-20250705-100100.pcap
```

---

# 十六、限制文件个数

例如

```bash
tcpdump \
-i eth0 \
-G 60 \
-W 10 \
-w capture-%H%M%S.pcap
```

表示：

* 每分钟一个文件
* 最多保存 10 个
* 超过后循环覆盖

---

# 十七、生产环境常用命令

### 抓某台服务器

```bash
tcpdump -i eth0 -nn host 10.1.1.100
```

### 抓 HTTPS

```bash
tcpdump -i eth0 -nn tcp port 443
```

### 抓 DNS

```bash
tcpdump -i eth0 -nn port 53
```

### 抓 MySQL

```bash
tcpdump -i eth0 -nn port 3306
```

### 抓 Redis

```bash
tcpdump -i eth0 -nn port 6379
```

### 抓 Kafka

```bash
tcpdump -i eth0 -nn port 9092
```

---

# 十八、抓包后用 Wireshark 分析

推荐抓包后保存为 `.pcap` 文件，再复制到本地使用 Wireshark 打开：

```bash
tcpdump -i eth0 -s 0 -nn -w capture.pcap
```

复制文件：

```bash
scp user@server:/tmp/capture.pcap .
```

然后用 Wireshark 查看 TCP 三次握手、TLS 握手、HTTP 请求、重传、延迟等信息。

---

# 十九、常用参数总结

| 参数             | 作用              |
| -------------- | --------------- |
| `-i eth0`      | 指定网卡            |
| `-i any`       | 抓所有网卡           |
| `-n`           | 不解析主机名          |
| `-nn`          | 不解析主机名和端口名      |
| `-s 0`         | 抓取完整数据包         |
| `-c 100`       | 抓取指定数量的数据包      |
| `-w file.pcap` | 保存到 pcap 文件     |
| `-r file.pcap` | 读取 pcap 文件      |
| `-A`           | 以 ASCII 显示数据    |
| `-X`           | 十六进制 + ASCII 显示 |
| `-C 100`       | 按文件大小滚动（MB）     |
| `-G 60`        | 按时间滚动（秒）        |
| `-W 10`        | 保留文件数量          |

## 推荐的抓包命令

在生产环境中，如果需要保留完整数据并便于后续分析，推荐使用以下命令：

```bash
sudo tcpdump -i eth0 -nn -s 0 -w capture.pcap
```

如果服务器流量较大，为避免生成单个超大文件，可采用滚动保存：

```bash
sudo tcpdump -i eth0 -nn -s 0 -C 100 -W 10 -w capture.pcap
```

该命令会在文件达到 **100 MB** 时自动切换到新文件，并最多保留 **10 个** 文件，适合长期抓包排查问题。
