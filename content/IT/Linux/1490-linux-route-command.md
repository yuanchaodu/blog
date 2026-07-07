---
title: Linux route 命令
section: IT
category: Linux
---

# Linux route 命令

<img src="images/Linux.svg" width="300">

`route` 命令用于查看和设置 Linux 的**路由表**，也就是系统决定“网络数据包往哪里走”的规则。

## 1. 查看路由表

```bash
route -n
```

常见输出：

```text
Destination     Gateway         Genmask         Flags Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    eth0
192.168.1.0     0.0.0.0         255.255.255.0   U     eth0
```

含义：

| 字段          | 说明    |
| ----------- | ----- |
| Destination | 目标网络  |
| Gateway     | 下一跳网关 |
| Genmask     | 子网掩码  |
| Flags       | 路由标志  |
| Iface       | 出口网卡  |

常见 Flags：

| 标志 | 含义      |
| -- | ------- |
| U  | 路由可用    |
| G  | 使用网关    |
| H  | 目标是单台主机 |

## 2. 添加默认网关

```bash
route add default gw 192.168.1.1
```

表示：访问非本地网段时，默认走 `192.168.1.1`。

## 3. 删除默认网关

```bash
route del default gw 192.168.1.1
```

## 4. 添加到某个网段的路由

```bash
route add -net 10.10.0.0 netmask 255.255.0.0 gw 192.168.1.254
```

表示：访问 `10.10.0.0/16` 网段时，走网关 `192.168.1.254`。

## 5. 删除网段路由

```bash
route del -net 10.10.0.0 netmask 255.255.0.0
```

## 6. 添加主机路由

```bash
route add -host 10.10.10.5 gw 192.168.1.254
```

表示：访问单台主机 `10.10.10.5` 时，走 `192.168.1.254`。

## 7. 现在更推荐用 ip route

`route` 命令比较老，现在很多系统推荐用：

```bash
ip route
```

添加默认网关：

```bash
ip route add default via 192.168.1.1
```

添加网段路由：

```bash
ip route add 10.10.0.0/16 via 192.168.1.254
```

查看路由：

```bash
ip route show
```

一句话理解：`route` 就像网络的“导航规则表”，告诉 Linux 去不同网络时该从哪个网卡、哪个网关出去。