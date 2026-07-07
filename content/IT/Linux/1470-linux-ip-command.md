---
title: Linux ip 命令
section: IT
category: Linux
---

# Linux ip 命令

<img src="images/Linux.svg" width="300">

`ip` 是 Linux 中最常用的网络管理命令之一，用来查看和配置**网卡、IP 地址、路由和邻居表（ARP/NDP）**。它属于 `iproute2` 工具集，基本已经取代 `ifconfig`、`route` 和 `arp` 等传统命令。

最常用的命令可以先记住下面这些：

```bash
# 查看所有网卡和 IP 地址
ip addr
ip a

# 查看指定网卡
ip addr show eth0

# 只查看 IPv4 地址
ip -4 addr

# 查看网卡状态
ip link

# 启用网卡
sudo ip link set eth0 up

# 禁用网卡
sudo ip link set eth0 down

# 查看路由表
ip route
ip r

# 查看访问某个地址实际走哪条路由
ip route get 8.8.8.8

# 查看邻居表（类似 arp -a）
ip neigh
```

其中最实用的是这三个：

```bash
ip a       # 我有哪些 IP 地址？
ip r       # 我的默认网关和路由是什么？
ip route get 8.8.8.8
           # 访问目标地址会从哪个网卡、哪个源 IP 出去？
```

例如：

```bash
$ ip route get 8.8.8.8

8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100
```

可以理解为：访问 `8.8.8.8` 时，数据经过网关 `192.168.1.1`，从 `eth0` 网卡发出，使用源地址 `192.168.1.100`。

如果你是在排查 Linux 网络问题，建议按这个顺序检查：

```bash
ip link          # 1. 网卡是否启用
ip addr          # 2. 是否获得 IP 地址
ip route         # 3. 是否有默认路由
ip neigh         # 4. 是否能找到网关的 MAC 地址
ip route get 目标IP   # 5. 实际会走哪条路由
```

你如果需要，我也可以继续整理一份 **Linux `ip` 命令完整速查表**，包括 `addr`、`link`、`route`、`neigh` 和多网卡路由的实际案例。