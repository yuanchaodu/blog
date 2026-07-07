---
title: Linux 网络基础
section: IT
category: Linux
---

# Linux 网络基础

<img src="images/Linux.svg" width="300">

本教程面向 Linux 初学者，也适合系统运维、网络管理和信息化人员。目标不是记住大量命令，而是建立一套清晰的思路：

> **网卡 → IP 地址 → 路由 → DNS → 端口 → 防火墙 → 抓包排查**

现代 Linux 网络管理应优先学习 `ip`、`ss`、`nmcli`、`nft` 等工具。`ip` 命令可以管理网卡、地址、路由和隧道；`nftables` 是现代 Linux 的主要包过滤框架。([Linux内核文档][1])

---

## 一、先理解 Linux 网络的整体结构

可以把一台 Linux 服务器想象成一栋办公楼：

| 网络概念   | 形象比喻       |
| ------ | ---------- |
| 网卡     | 楼的大门       |
| MAC 地址 | 大门的硬件编号    |
| IP 地址  | 楼的地址       |
| 子网掩码   | 判断是不是同一个园区 |
| 默认网关   | 通往外部的出口    |
| 路由表    | 导航地图       |
| DNS    | 通讯录        |
| 端口     | 房间号        |
| 防火墙    | 门卫         |
| Socket | 程序使用的通信窗口  |

一台 Linux 主机访问网络，大致经过下面的过程：

```text
应用程序
   ↓
TCP / UDP
   ↓
IP 路由判断
   ↓
网络接口
   ↓
交换机 / 路由器
   ↓
目标主机
```

Linux 内核负责网络协议栈、路由、网卡、包过滤等底层能力。用户空间中的 `ip`、NetworkManager、systemd-networkd 等工具，则负责查看或修改这些配置。([Linux内核文档][1])

---

# 二、第一层：认识网络接口

## 1. 查看所有网卡

```bash
ip link
```

常见输出：

```text
1: lo: <LOOPBACK,UP>
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP>
```

常见接口包括：

| 接口        | 含义           |
| --------- | ------------ |
| `lo`      | 本地回环接口       |
| `eth0`    | 传统以太网接口名     |
| `ens192`  | 常见服务器网卡名     |
| `enp0s3`  | 常见 Linux 网卡名 |
| `wlan0`   | 无线网卡         |
| `docker0` | Docker 虚拟网桥  |
| `br0`     | Linux 网桥     |

其中：

```text
UP
```

表示网卡已启用。

```text
LOWER_UP
```

通常表示底层链路正常。

## 2. 启用和关闭网卡

```bash
sudo ip link set ens192 up
```

关闭：

```bash
sudo ip link set ens192 down
```

现代 Linux 中，`ip` 命令是查看和操作网络设备、地址及路由的核心工具。相比旧的 `ifconfig` 和 `route`，更建议优先掌握 `ip`。([man7.org][2])

---

# 三、第二层：理解 IP 地址

## 1. 查看 IP 地址

推荐使用：

```bash
ip addr
```

简写：

```bash
ip a
```

只查看 IPv4：

```bash
ip -4 addr
```

假设看到：

```text
inet 192.168.10.25/24
```

它包含两个部分：

```text
192.168.10.25    IP 地址
/24              子网前缀
```

`/24` 等价于：

```text
255.255.255.0
```

因此，这台机器通常所在的网段是：

```text
192.168.10.0/24
```

---

## 2. 什么是子网

假设有两台机器：

```text
A：192.168.10.25/24
B：192.168.10.50/24
```

它们通常可以直接通信。

但如果是：

```text
A：192.168.10.25/24
B：192.168.20.50/24
```

它们属于不同网段，通常需要经过路由器。

可以简单理解为：

> 同一个小区，可以直接串门；不同小区，需要经过道路和路口。

---

## 3. 临时添加 IP 地址

```bash
sudo ip addr add 192.168.10.100/24 dev ens192
```

删除：

```bash
sudo ip addr del 192.168.10.100/24 dev ens192
```

注意，这类命令通常直接修改当前运行状态。机器重启后是否保留，取决于发行版和网络管理方式。因此，生产服务器不能只依赖临时命令。([红帽文档][3])

---

# 四、第三层：理解路由

路由是 Linux 网络中非常重要的概念。

## 1. 查看路由表

```bash
ip route
```

常见输出：

```text
default via 192.168.10.1 dev ens192

192.168.10.0/24 dev ens192
```

第一条：

```text
default via 192.168.10.1
```

表示：

> 如果不知道数据应该往哪里发，就交给 192.168.10.1。

这个地址通常就是默认网关。

第二条：

```text
192.168.10.0/24 dev ens192
```

表示：

> 访问 192.168.10.0/24 网段时，从 ens192 网卡发送。

---

## 2. 路由表像导航软件

例如访问：

```text
192.168.10.50
```

Linux 发现它属于：

```text
192.168.10.0/24
```

于是直接通过本地网卡发送。

如果访问：

```text
8.8.8.8
```

没有更具体的路由，就走：

```text
default
```

因此，Linux 选择路由时，可以先简单理解为：

> **优先走更具体的路，实在找不到，再走默认路由。**

---

## 3. 查看访问目标会走哪条路

这是非常实用的命令：

```bash
ip route get 8.8.8.8
```

它可以帮助判断：

* 使用哪个网卡；
* 使用哪个源 IP；
* 经过哪个网关。

网络故障排查时，这条命令很有价值。

---

# 五、第四层：理解 DNS

用户访问网站时，通常输入的是域名：

```text
www.example.com
```

计算机真正通信时，需要 IP 地址。

DNS 的作用就是：

```text
域名 → IP 地址
```

可以把 DNS 理解为网络世界的“通讯录”。

---

## 1. 查看 DNS 配置

```bash
cat /etc/resolv.conf
```

可能看到：

```text
nameserver 192.168.10.1
```

部分现代 Linux 系统使用 systemd-resolved，可以查看：

```bash
resolvectl status
```

---

## 2. 查询域名

常用命令：

```bash
nslookup example.com
```

更推荐掌握：

```bash
dig example.com
```

只查看结果：

```bash
dig +short example.com
```

---

## 3. 判断是不是 DNS 故障

假设：

```bash
ping 8.8.8.8
```

能够成功，但：

```bash
ping example.com
```

失败。

这时应优先检查 DNS。

这个判断非常重要：

```text
IP 能通，域名不通
        ↓
优先检查 DNS
```

---

# 六、第五层：理解端口和 Socket

一台服务器只有一个 IP，但可以运行很多服务。

例如：

| 服务    | 常见端口 |
| ----- | ---: |
| SSH   |   22 |
| HTTP  |   80 |
| HTTPS |  443 |
| DNS   |   53 |

IP 地址负责找到机器，端口负责找到程序。

可以理解为：

```text
IP 地址 = 办公楼地址
端口号 = 房间号
```

---

## 1. 查看监听端口

推荐：

```bash
ss -lntp
```

参数含义：

```text
-l    只看监听状态
-n    不解析名称
-t    TCP
-p    显示进程
```

查看 UDP：

```bash
ss -lnup
```

查看所有连接：

```bash
ss -antp
```

---

## 2. 判断服务是否监听

例如检查 8080 端口：

```bash
ss -lntp | grep 8080
```

如果没有结果，通常说明：

* 服务没有启动；
* 服务启动失败；
* 服务没有监听该端口。

如果看到：

```text
127.0.0.1:8080
```

表示只能本机访问。

如果看到：

```text
0.0.0.0:8080
```

表示监听所有 IPv4 地址。

---

# 七、第六层：掌握基本连通性测试

## 1. ping：检查基础连通性

```bash
ping 192.168.10.1
```

只发送 4 次：

```bash
ping -c 4 192.168.10.1
```

它主要用于判断：

* 主机是否可能可达；
* 延迟大概是多少；
* 是否存在丢包。

但要注意：

> ping 不通，不一定代表服务不通。

因为部分设备会禁止 ICMP。

---

## 2. traceroute：查看经过的路径

```bash
traceroute 8.8.8.8
```

或者：

```bash
tracepath 8.8.8.8
```

它适合排查：

```text
数据到底在哪一跳出现问题？
```

---

## 3. curl：检查应用服务

检查 HTTP：

```bash
curl http://192.168.10.100
```

查看详细过程：

```bash
curl -v https://example.com
```

只查看响应头：

```bash
curl -I https://example.com
```

---

## 4. nc：测试端口

例如测试服务器 443 端口：

```bash
nc -vz 192.168.10.100 443
```

这比单纯使用 `ping` 更接近实际业务。

例如：

```text
ping 成功
nc 失败
```

说明主机可能在线，但目标端口存在问题。

---

# 八、第七层：理解 ARP 和邻居表

同一局域网内，IP 数据最终还需要找到目标网卡。

IPv4 中常涉及：

```text
IP 地址 → MAC 地址
```

查看邻居表：

```bash
ip neigh
```

可能看到：

```text
192.168.10.1 dev ens192 lladdr 00:11:22:33:44:55 REACHABLE
```

这表示 Linux 已经知道：

```text
192.168.10.1
```

对应哪个 MAC 地址。

如果同网段设备无法访问，可以检查：

```bash
ip neigh
```

常见状态包括：

```text
REACHABLE
STALE
FAILED
INCOMPLETE
```

如果大量出现 `FAILED` 或 `INCOMPLETE`，可能存在二层网络、VLAN、交换机或地址配置问题。

---

# 九、第八层：理解防火墙

Linux 内核中的 Netfilter 提供包过滤、NAT 等能力；现代规则管理通常使用 `nftables`。Ubuntu 的安全文档也将 `nftables` 描述为现代的包分类和过滤框架。([Netfilter][4])

查看规则：

```bash
sudo nft list ruleset
```

实际环境中，还可能遇到：

```text
firewalld
ufw
iptables
nftables
```

它们之间不要简单理解为完全独立的四套东西。

更准确地说：

```text
Linux 内核
    ↓
Netfilter
    ↓
nftables / iptables
    ↓
firewalld 等管理工具
```

在现代 Linux 学习中，建议逐步把重点放在 `nftables` 上。([红帽文档][5])

---

# 十、最重要的网络故障排查方法

遇到“网络不通”，不要随机输入命令。

建议按下面顺序检查：

1. **网卡是否正常**

```bash
ip link
```

2. **IP 地址是否正确**

```bash
ip addr
```

3. **路由是否正确**

```bash
ip route
```

4. **目标走哪条路**

```bash
ip route get 目标IP
```

5. **本地网关是否可达**

```bash
ping 网关IP
```

6. **远程 IP 是否可达**

```bash
ping 目标IP
```

7. **DNS 是否正常**

```bash
dig 域名
```

8. **目标端口是否可达**

```bash
nc -vz 目标IP 端口
```

9. **本机服务是否监听**

```bash
ss -lntp
```

10. **防火墙是否阻止**

```bash
sudo nft list ruleset
```

这套方法的核心是：

> **从底层向上检查，从本机向远端检查。**

不要一开始就怀疑应用程序，也不要一开始就关闭防火墙。

---

# 十一、建议的学习路线

对于信息化和系统运维工作，建议分四个阶段学习。

**第一阶段：基础网络**

掌握：

```text
IP 地址
子网
网关
路由
DNS
TCP
UDP
端口
```

**第二阶段：Linux 常用命令**

重点练习：

```text
ip link
ip addr
ip route
ip neigh
ss
ping
tracepath
dig
curl
nc
```

**第三阶段：服务器网络管理**

继续学习：

```text
NetworkManager
nmcli
systemd-networkd
nftables
VLAN
Bond
Bridge
```

企业 Linux 网络配置通常还会涉及 VLAN、链路聚合、网桥和隧道等能力。([红帽文档][6])

**第四阶段：高级网络**

最后再学习：

```text
Network Namespace
veth
容器网络
策略路由
NAT
抓包分析
eBPF 网络观测
```

Network Namespace 可以隔离网络设备、协议栈、路由表、防火墙规则和端口等网络资源，是理解容器网络的重要基础。([man7.org][7])

---

## 一张图记住 Linux 网络

```text
应用程序
   │
   │ 端口
   ▼
TCP / UDP
   │
   │ Socket
   ▼
IP 协议
   │
   ├── DNS：名字翻译成 IP
   │
   ├── 路由表：决定往哪里走
   │
   ├── 防火墙：决定是否放行
   │
   ▼
网络接口
   │
   ├── IP 地址
   ├── MAC 地址
   └── ARP / 邻居表
   │
   ▼
交换机 / 路由器
   │
   ▼
目标服务器
```

建议下一步直接进入 **《Linux 网络基础教程（二）：IP 地址、子网掩码与 CIDR 详解》**。这一部分是整个 Linux 网络知识体系的地基。后续可以按“原理讲解 + 实验环境 + 操作命令 + 故障案例”的方式继续学习。

[1]: https://docs.kernel.org/networking/index.html "Networking"
[2]: https://man7.org/linux/man-pages/man8/ip.8.html "ip(8) - Linux manual page"
[3]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_ip_commands "3.6. Configuring IP Networking with ip Commands"
[4]: https://www.netfilter.org/ "netfilter/iptables project homepage - The netfilter.org project"
[5]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-nftables_configuring-and-managing-networking "Chapter 42. Getting started with nftables"
[6]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/index "Configuring and managing networking"
[7]: https://man7.org/linux/man-pages/man7/network_namespaces.7.html "network_namespaces(7) - Linux manual page"