---
title: Linux ping 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4Antmu
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/155'
---

# Linux ping 命令

<img src="images/Linux.svg" width="300">

`ping` 是 Linux 中最常用的网络诊断命令之一，用于测试两台设备之间的网络连通性，并测量网络延迟（RTT，Round Trip Time）。

## 一、基本语法

```bash
ping [选项] 主机名或IP地址
```

例如：

```bash
ping 192.168.1.1
```

或

```bash
ping www.baidu.com
```

输出示例：

```text
PING www.baidu.com (39.156.66.10) 56(84) bytes of data.
64 bytes from 39.156.66.10: icmp_seq=1 ttl=52 time=12.3 ms
64 bytes from 39.156.66.10: icmp_seq=2 ttl=52 time=11.9 ms
64 bytes from 39.156.66.10: icmp_seq=3 ttl=52 time=12.1 ms
```

按 **Ctrl + C** 可以停止。

---

## 二、工作原理

`ping` 基于 **ICMP（Internet Control Message Protocol）**。

工作流程如下：

```
本机
 │
 │  ICMP Echo Request（请求）
 ▼
目标主机
 │
 │  ICMP Echo Reply（应答）
 ▼
本机
```

如果收到应答，说明网络可达。

---

## 三、输出结果说明

例如：

```text
64 bytes from 192.168.1.1:
icmp_seq=5 ttl=64 time=0.35 ms
```

各字段含义：

| 字段           | 说明         |
| ------------ | ---------- |
| 64 bytes     | 收到的数据包大小   |
| icmp_seq=5   | 第5个ICMP数据包 |
| ttl=64       | TTL（生存时间）  |
| time=0.35 ms | 往返时间（RTT）  |

---

## 四、常用参数

### 1. 指定发送次数 `-c`

发送指定次数后自动结束。

```bash
ping -c 4 8.8.8.8
```

输出：

```text
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
```

---

### 2. 指定时间间隔 `-i`

默认每秒发送一次。

例如每2秒发送一次：

```bash
ping -i 2 192.168.1.1
```

> 普通用户一般不能设置小于 0.2 秒，root 可以。

---

### 3. 指定超时时间 `-W`

等待响应时间（秒）。

```bash
ping -W 3 192.168.1.1
```

表示等待3秒，没有回复就认为超时。

---

### 4. 设置总超时时间 `-w`

例如：

```bash
ping -w 10 8.8.8.8
```

10秒后自动退出。

---

### 5. 指定数据包大小 `-s`

默认56字节。

例如发送1024字节：

```bash
ping -s 1024 192.168.1.1
```

---

### 6. 不解析域名 `-n`

默认会解析IP对应主机名。

```bash
ping -n 8.8.8.8
```

速度会更快一些。

---

### 7. 指定网络接口 `-I`

例如：

```bash
ping -I eth0 192.168.1.1
```

或者：

```bash
ping -I 192.168.1.100 192.168.1.1
```

表示指定源网卡或源IP。

---

### 8. 广播Ping（不常用）

```bash
ping -b 192.168.1.255
```

需要系统允许广播。

---

### 9. Flood模式（压力测试）

```bash
sudo ping -f 192.168.1.1
```

高速连续发送数据包。

> 仅建议在受控环境中使用，容易造成网络拥塞。

---

## 五、统计信息

结束后会显示：

```text
--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss
round-trip min/avg/max/mdev = 0.342/0.421/0.513/0.067 ms
```

含义：

| 字段                  | 说明         |
| ------------------- | ---------- |
| packets transmitted | 发送包数       |
| received            | 收到包数       |
| packet loss         | 丢包率        |
| min                 | 最小延迟       |
| avg                 | 平均延迟       |
| max                 | 最大延迟       |
| mdev                | 延迟抖动（标准偏差） |

例如：

```
packet loss = 20%
```

说明：

* 网络不稳定
* 存在丢包
* 可能交换机、路由器、网线存在问题

---

## 六、常见故障分析

### 1. `Destination Host Unreachable`

```text
From 192.168.1.100 icmp_seq=1 Destination Host Unreachable
```

表示：

* 目标不存在
* 路由不可达
* 网关配置错误

---

### 2. `Request timeout`

```text
Request timeout for icmp_seq 3
```

说明：

* 对方没有回应
* 网络中断
* 防火墙禁止ICMP
* 主机关闭

---

### 3. `Unknown host`

```text
ping: unknown host www.abc.com
```

说明：

DNS解析失败。

可以测试：

```bash
ping 8.8.8.8
```

如果可以，而：

```bash
ping www.baidu.com
```

不可以，则通常说明DNS配置有问题。

---

## 七、典型使用场景

### 检查本机网络

```bash
ping 127.0.0.1
```

测试TCP/IP协议栈是否正常。

---

### 检查网卡

```bash
ping 本机IP
```

例如：

```bash
ping 192.168.1.100
```

确认本机网络接口工作正常。

---

### 检查网关

```bash
ping 192.168.1.1
```

若失败，可能是：

* 网线异常
* VLAN配置错误
* 网关故障

---

### 检查外网

```bash
ping 8.8.8.8
```

若失败：

* 默认网关错误
* NAT异常
* 外网不可达

---

### 检查DNS

```bash
ping www.baidu.com
```

若：

```
ping 8.8.8.8
```

成功，而：

```
ping www.baidu.com
```

失败，则说明DNS解析可能存在问题。

---

## 八、Linux 与 Windows `ping` 的区别

| 功能      | Linux                | Windows           |
| ------- | -------------------- | ----------------- |
| 默认发送次数  | 持续发送，直到按 `Ctrl+C` 停止 | 默认发送4次            |
| 指定次数    | `-c`                 | `-n`              |
| 发送间隔    | `-i`                 | 固定约1秒（无直接参数）      |
| 包大小     | `-s`                 | `-l`              |
| TTL设置   | `-t`                 | `-i`              |
| 禁止DNS解析 | `-n`                 | `-a` 为反向解析，默认行为不同 |

## 九、实用示例

```bash
# 连续测试目标主机
ping 192.168.1.10

# 测试4次后退出
ping -c 4 192.168.1.10

# 每2秒发送一次，共5次
ping -i 2 -c 5 192.168.1.10

# 指定数据包大小为1000字节
ping -s 1000 192.168.1.10

# 设置单次响应超时时间为3秒
ping -W 3 -c 4 192.168.1.10

# 指定网卡发送
ping -I eth0 192.168.1.10

# 不进行DNS反向解析
ping -n 8.8.8.8
```

## 十、排查网络故障的推荐顺序

当遇到网络连接问题时，可以按以下顺序使用 `ping`：

1. **检查协议栈**：`ping 127.0.0.1`
2. **检查本机网卡**：`ping 本机IP`
3. **检查默认网关**：`ping 网关IP`
4. **检查外网 IP**：`ping 8.8.8.8`
5. **检查域名解析**：`ping www.baidu.com`

如果前一步失败，就优先定位该层的问题；如果前一步成功而后一步失败，则问题通常出现在两者之间。这种由近及远的排查方法能够快速定位故障所在。
