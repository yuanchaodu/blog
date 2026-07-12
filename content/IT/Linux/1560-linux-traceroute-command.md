---
title: Linux traceroute 命令
section: IT
category: Linux
---

# Linux traceroute 命令

<img src="images/Linux.svg" width="300">

`traceroute` 是 Linux 中用于**跟踪数据包从本机到目标主机所经过路由路径**的命令。相比 `ping` 只能告诉你目标是否可达，`traceroute` 能告诉你**数据包经过了哪些路由器、每一跳的延迟是多少，以及在哪一跳出现了问题**。

---

# 一、基本语法

```bash
traceroute [选项] 主机名或IP地址
```

例如：

```bash
traceroute 8.8.8.8
```

或者

```bash
traceroute www.baidu.com
```

如果系统提示没有该命令，可以安装：

Ubuntu / Debian：

```bash
sudo apt install traceroute
```

CentOS / RHEL：

```bash
sudo yum install traceroute
```

Rocky / AlmaLinux：

```bash
sudo dnf install traceroute
```

---

# 二、工作原理

`traceroute` 利用了 **IP 数据包中的 TTL（Time To Live，生存时间）** 字段。

工作过程如下：

1. 发送 TTL=1 的数据包。
2. 第一跳路由器收到后，TTL 减为 0，丢弃数据包，并返回 **ICMP Time Exceeded** 报文。
3. `traceroute` 记录第一跳路由器的 IP 地址和响应时间。
4. 再发送 TTL=2 的数据包。
5. 第二跳返回 ICMP。
6. 重复上述过程，直到到达目标主机。

示意图：

```text
本机
 │ TTL=1
 ▼
R1（返回ICMP）
 │ TTL=2
 ▼
R2（返回ICMP）
 │ TTL=3
 ▼
R3（返回ICMP）
 │
 ▼
目标服务器（返回响应）
```

因此，`traceroute` 可以逐跳显示网络路径。

---

# 三、输出示例

例如：

```text
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max

 1  192.168.1.1      0.35 ms  0.42 ms  0.39 ms
 2  10.10.0.1        3.21 ms  3.18 ms  3.25 ms
 3  202.96.134.1    10.24 ms 10.36 ms 10.12 ms
 4  219.158.8.201   15.53 ms 15.31 ms 15.48 ms
 5  8.8.8.8         18.65 ms 18.42 ms 18.71 ms
```

各字段说明：

| 字段          | 说明          |
| ----------- | ----------- |
| 1、2、3       | 第几跳（Hop）    |
| 192.168.1.1 | 该跳路由器 IP 地址 |
| 0.35 ms     | 第一次探测耗时     |
| 0.42 ms     | 第二次探测耗时     |
| 0.39 ms     | 第三次探测耗时     |

默认情况下，每一跳会发送 **3 个探测包**。

---

# 四、常用参数

## 1. 指定最大跳数 `-m`

默认最大 30 跳。

例如：

```bash
traceroute -m 20 8.8.8.8
```

最多探测 20 跳。

---

## 2. 不解析主机名 `-n`

推荐在排查网络时使用。

```bash
traceroute -n 8.8.8.8
```

输出更快，例如：

```text
1 192.168.1.1
2 10.10.0.1
3 202.96.134.1
```

避免 DNS 解析影响结果。

---

## 3. 指定等待时间 `-w`

例如：

```bash
traceroute -w 3 8.8.8.8
```

等待 3 秒后超时。

---

## 4. 指定探测次数 `-q`

默认每跳发送 3 个包。

例如：

```bash
traceroute -q 5 8.8.8.8
```

每跳发送 5 个探测包。

---

## 5. 指定数据包大小

例如：

```bash
traceroute 8.8.8.8 1200
```

发送 1200 字节的数据包。

---

## 6. 使用 ICMP 模式 `-I`

Linux 默认使用 **UDP** 探测，而有些设备可能屏蔽 UDP，可改用 ICMP：

```bash
sudo traceroute -I 8.8.8.8
```

这与 Windows 的 `tracert` 更接近。

---

## 7. 使用 TCP 模式 `-T`

某些网络环境下 UDP 和 ICMP 被过滤，而 TCP 可正常通过。

例如：

```bash
sudo traceroute -T -p 443 www.baidu.com
```

参数说明：

* `-T`：TCP 探测。
* `-p 443`：目标端口为 443（HTTPS）。

---

# 五、特殊符号说明

## 出现 `*`

例如：

```text
5 * * *
```

表示该跳没有回应。

可能原因：

* 路由器禁用了 ICMP。
* 防火墙丢弃探测包。
* 网络拥塞导致超时。

如果后续跳数还能正常显示，通常只是该路由器不响应探测，并不代表网络中断。

---

## 一直都是 `*`

例如：

```text
5 * * *
6 * * *
7 * * *
```

可能原因：

* 网络中断。
* 路由故障。
* 目标主机不可达。
* 中间设备屏蔽了探测报文。

---

# 六、Linux 与 Windows 的区别

| 功能       | Linux `traceroute` | Windows `tracert` |
| -------- | ------------------ | ----------------- |
| 默认协议     | UDP                | ICMP Echo         |
| 可选协议     | UDP、ICMP、TCP       | 主要为 ICMP          |
| 默认最大跳数   | 30                 | 30                |
| 默认每跳探测次数 | 3                  | 3                 |
| DNS 解析关闭 | `-n`               | `-d`              |

---

# 七、常见应用场景

### 1. 查找网络中断位置

```bash
traceroute www.baidu.com
```

如果输出：

```text
1 192.168.1.1
2 10.10.0.1
3 *
4 *
```

说明故障可能发生在第二跳之后。

---

### 2. 判断出口线路

```bash
traceroute 114.114.114.114
```

可以查看：

* 是否经过运营商骨干网。
* 是否经过跨地区链路。
* 是否经过国际出口。

---

### 3. 判断延迟增加的位置

例如：

```text
1   0.3 ms
2   2 ms
3   3 ms
4   85 ms
5   90 ms
```

可以看出：

* 第 4 跳开始延迟明显增加。
* 高延迟可能由该路由器或其后的链路引起。

---

### 4. 检查跨运营商路由

例如：

```text
电信
   │
联通
   │
移动
   │
目标服务器
```

可以分析不同运营商之间的互联情况，辅助排查跨网访问性能问题。

---

# 八、`traceroute` 与 `ping` 的区别

| 对比项    | `ping`                  | `traceroute`        |
| ------ | ----------------------- | ------------------- |
| 主要功能   | 测试目标是否可达                | 查看数据包经过的完整路径        |
| 使用协议   | ICMP Echo Request/Reply | 默认 UDP，也支持 ICMP、TCP |
| 显示路径   | 否                       | 是                   |
| 显示每跳延迟 | 否                       | 是                   |
| 适合排查   | 网络是否连通、丢包、时延            | 定位故障发生在哪一跳          |

---

# 九、`mtr`：更强大的替代工具

如果需要持续监测链路质量，推荐使用 **`mtr`（My Traceroute）**。它结合了 `ping` 和 `traceroute` 的优点，能够实时显示每一跳的：

* 丢包率（Loss%）
* 已发送探测包数量（Snt）
* 最后一次延迟（Last）
* 平均延迟（Avg）
* 最佳延迟（Best）
* 最差延迟（Wrst）
* 延迟波动（StDev）

安装示例：

```bash
sudo apt install mtr      # Debian/Ubuntu
sudo dnf install mtr      # Rocky/AlmaLinux/Fedora
sudo yum install mtr      # CentOS
```

运行：

```bash
mtr 8.8.8.8
```

生成报告模式（适合保存或分享）：

```bash
mtr -r -c 100 8.8.8.8
```

其中：

* `-r`：报告模式（运行结束后输出统计结果）。
* `-c 100`：发送 100 次探测。

在企业网络故障排查中，`mtr` 通常比 `traceroute` 更有价值，因为它不仅能显示路径，还能持续统计每一跳的丢包率和时延变化，更容易定位间歇性网络问题。