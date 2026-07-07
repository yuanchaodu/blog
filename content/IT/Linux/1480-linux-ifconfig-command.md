---
title: Linux ifconfig 命令
section: IT
category: Linux
---

# Linux ifconfig 命令

<img src="images/Linux.svg" width="300">

`ifconfig` 是 Linux 中常见的网络配置命令，主要用于**查看和临时配置网络接口**。

## 1. 查看所有已启用的网络接口

```bash
ifconfig
```

常见输出：

```text
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>
        inet 192.168.1.100
        netmask 255.255.255.0
        ether 00:11:22:33:44:55
```

重点字段如下：

| 字段             | 含义      |
| -------------- | ------- |
| `eth0`、`ens33` | 网卡名称    |
| `inet`         | IPv4 地址 |
| `inet6`        | IPv6 地址 |
| `netmask`      | 子网掩码    |
| `ether`        | MAC 地址  |
| `RX`           | 接收的数据   |
| `TX`           | 发送的数据   |
| `UP`           | 网卡已启用   |
| `RUNNING`      | 网卡正在运行  |

## 2. 查看指定网卡

```bash
ifconfig eth0
```

如果网卡名称是 `ens33`：

```bash
ifconfig ens33
```

不知道网卡名称时，可以先执行：

```bash
ifconfig -a
```

`-a` 会显示所有网络接口，包括未启用的接口。

## 3. 启用或关闭网卡

启用：

```bash
sudo ifconfig eth0 up
```

关闭：

```bash
sudo ifconfig eth0 down
```

远程连接服务器时要谨慎执行 `down`，否则可能直接断开 SSH 连接。

## 4. 临时设置 IP 地址

```bash
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```

这种设置通常是**临时的**，系统重启后可能失效。

## 5. 新版 Linux 更推荐使用 `ip`

现在很多新版 Linux 发行版已经不再默认安装 `ifconfig`，更推荐使用 `ip` 命令：

| `ifconfig` 命令        | 推荐的新命令                  |
| -------------------- | ----------------------- |
| `ifconfig`           | `ip addr`               |
| `ifconfig -a`        | `ip link`               |
| `ifconfig eth0`      | `ip addr show eth0`     |
| `ifconfig eth0 up`   | `ip link set eth0 up`   |
| `ifconfig eth0 down` | `ip link set eth0 down` |

日常排查网络问题，我更建议记住这三个命令：

```bash
ip addr
ip route
ip link
```

它们分别用于查看 **IP 地址、路由表和网卡状态**。如果你是在处理具体的 `ifconfig` 输出，也可以把输出贴出来，我可以逐项帮你解释。