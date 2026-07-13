---
title: Linux 网络故障排查
section: IT
category: Linux
---

# Linux 网络故障排查

<img src="images/Linux.svg" width="300">

Linux 网络故障排查通常遵循**“由外到内、由下到上”**的思路，即先检查物理连接，再检查网络配置，最后检查应用服务。这样可以避免一开始就陷入复杂的问题分析。

下面介绍一套比较完整的排查方法。

---

# 一、确认故障现象

首先要明确是什么问题。

常见现象包括：

| 现象                   | 可能原因              |
| -------------------- | ----------------- |
| 无法访问任何网络             | 网卡、IP、路由、DNS      |
| 能 ping IP，不能 ping 域名 | DNS 故障            |
| 本机正常，别人无法访问本机        | 防火墙、端口未监听         |
| 某个网站无法访问             | 路由、代理、防火墙         |
| 网络时断时续               | MTU、网卡驱动、交换机、双工模式 |

先回答几个问题：

* 能不能 ping 本机？
* 能不能 ping 网关？
* 能不能 ping 外网 IP（例如 8.8.8.8）？
* 能不能 ping 域名（例如 [www.baidu.com）？](http://www.baidu.com）？)

这四步往往就能定位大方向。

---

# 二、检查网卡状态

查看网卡

```bash
ip link
```

或者

```bash
ifconfig
```

输出类似：

```
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP>
```

重点关注：

* UP
* LOWER_UP

如果看到

```
state DOWN
```

说明网卡没有启动：

```bash
sudo ip link set ens33 up
```

---

查看网卡信息：

```bash
ip addr
```

例如：

```
ens33:
    inet 192.168.1.100/24
```

如果没有

```
inet
```

说明没有获取 IP。

---

# 三、检查 IP 地址

查看：

```bash
ip addr
```

或者

```bash
hostname -I
```

确认：

* IP 是否正确
* 子网掩码是否正确

例如：

```
192.168.10.25/24
```

如果 IP 不正确：

DHCP：

```bash
dhclient ens33
```

或者：

```
nmcli connection up
```

静态 IP：

检查：

```
/etc/sysconfig/network-scripts/
```

（CentOS）

或

```
/etc/netplan/
```

（Ubuntu）

---

# 四、检查路由

查看路由：

```bash
ip route
```

例如：

```
default via 192.168.1.1 dev ens33
```

重点：

有没有默认路由：

```
default via
```

没有的话：

```bash
sudo ip route add default via 192.168.1.1
```

---

# 五、测试网络连通性

## ① Ping 本机

```bash
ping 127.0.0.1
```

失败：

说明 TCP/IP 协议栈异常。

---

## ② Ping 本机 IP

例如：

```bash
ping 192.168.1.100
```

失败：

说明网卡异常。

---

## ③ Ping 网关

例如：

```bash
ping 192.168.1.1
```

失败：

可能：

* 网线
* VLAN
* 交换机
* 网卡

---

## ④ Ping 外网 IP

例如：

```bash
ping 8.8.8.8
```

如果：

```
ping 网关成功
ping 外网失败
```

说明：

* 默认路由
* 出口网络

有问题。

---

## ⑤ Ping 域名

例如：

```bash
ping www.baidu.com
```

如果：

```
ping 8.8.8.8
```

成功

但是：

```
ping www.baidu.com
```

失败

说明 DNS 故障。

---

# 六、检查 DNS

查看：

```bash
cat /etc/resolv.conf
```

例如：

```
nameserver 114.114.114.114
nameserver 8.8.8.8
```

测试：

```bash
nslookup www.baidu.com
```

或者：

```bash
dig www.baidu.com
```

如果解析失败：

修改：

```
/etc/resolv.conf
```

例如：

```
nameserver 223.5.5.5
```

---

# 七、检查端口

查看监听：

```bash
ss -tulnp
```

或者：

```bash
netstat -tulnp
```

例如：

```
LISTEN 0 128 *:80
```

说明：

80 端口已监听。

如果没有：

说明服务没有启动。

---

查看连接：

```bash
ss -tan
```

查看：

```
ESTABLISHED
TIME_WAIT
CLOSE_WAIT
```

可以判断连接状态。

---

# 八、检查防火墙

CentOS：

```bash
firewall-cmd --list-all
```

Ubuntu：

```bash
ufw status
```

iptables：

```bash
iptables -L -n
```

查看：

是否拒绝：

```
DROP
REJECT
```

测试：

临时关闭：

```bash
systemctl stop firewalld
```

或者：

```bash
ufw disable
```

注意：生产环境不要长时间关闭防火墙，可临时测试后恢复。

---

# 九、检查 SELinux

查看：

```bash
getenforce
```

输出：

```
Enforcing
```

可能限制网络访问。

临时：

```bash
setenforce 0
```

查看是否恢复。

---

# 十、检查 NetworkManager

查看：

```bash
systemctl status NetworkManager
```

查看网卡：

```bash
nmcli device
```

查看连接：

```bash
nmcli connection show
```

重启：

```bash
systemctl restart NetworkManager
```

---

# 十一、查看网络统计

查看接口：

```bash
ip -s link
```

例如：

```
RX errors
TX errors
dropped
```

如果不断增长：

可能：

* 网卡故障
* 双工不匹配
* MTU

---

查看速度：

```bash
ethtool ens33
```

例如：

```
Speed: 1000Mb/s
Duplex: Full
```

查看是否：

```
Half Duplex
```

容易造成网络性能问题。

---

# 十二、查看 ARP

查看：

```bash
ip neigh
```

例如：

```
192.168.1.1 dev ens33 lladdr ...
```

如果：

```
FAILED
```

说明 ARP 无法解析。

可以：

```bash
arp -n
```

---

# 十三、检查路由路径

查看：

```bash
traceroute 8.8.8.8
```

或者：

```bash
tracepath 8.8.8.8
```

判断：

哪一跳中断。

---

# 十四、抓包分析

Linux 最重要工具：

```bash
tcpdump
```

例如：

抓 ICMP：

```bash
tcpdump -i ens33 icmp
```

抓 HTTP：

```bash
tcpdump -i ens33 port 80
```

抓 DNS：

```bash
tcpdump -i ens33 port 53
```

保存：

```bash
tcpdump -i ens33 -w test.pcap
```

然后：

使用 Wireshark 分析。

---

# 十五、查看系统日志

查看：

```bash
journalctl -u NetworkManager
```

或者：

```bash
journalctl -xe
```

内核日志：

```bash
dmesg | grep -i eth
```

查看：

* 网卡驱动
* Link Down
* Link Up
* DHCP

等异常。

---

# 十六、常用排查命令速查

| 排查内容   | 常用命令                            |
| ------ | ------------------------------- |
| 网卡状态   | `ip link`                       |
| IP 地址  | `ip addr`                       |
| 路由     | `ip route`                      |
| 连通性    | `ping`                          |
| DNS 解析 | `nslookup`、`dig`                |
| 路由路径   | `traceroute`、`tracepath`        |
| 网络连接   | `ss -tan`                       |
| 监听端口   | `ss -tulnp`                     |
| 抓包     | `tcpdump`                       |
| ARP 表  | `ip neigh`                      |
| 网卡统计   | `ip -s link`                    |
| 网卡参数   | `ethtool`                       |
| 网络管理   | `nmcli`                         |
| 防火墙    | `firewall-cmd`、`iptables`、`ufw` |
| 日志     | `journalctl`、`dmesg`            |

## 推荐的排查流程（适用于大多数故障）

按照 **OSI 模型** 和 **“先基础、后应用”** 的原则，可以使用以下流程：

1. **物理层**：检查网线、交换机端口、网卡状态（`ip link`、`ethtool`）。
2. **网络配置**：检查 IP 地址（`ip addr`）、默认路由（`ip route`）、ARP（`ip neigh`）。
3. **连通性测试**：依次执行 `ping 127.0.0.1` → `ping 本机IP` → `ping 网关` → `ping 外网IP` → `ping 域名`。
4. **名称解析**：检查 DNS 配置（`cat /etc/resolv.conf`、`dig`、`nslookup`）。
5. **传输层**：检查端口监听和连接状态（`ss -tulnp`、`ss -tan`）。
6. **安全策略**：检查防火墙（`firewall-cmd`、`iptables`、`ufw`）和 SELinux（`getenforce`）。
7. **深入分析**：查看日志（`journalctl`、`dmesg`），必要时使用 `tcpdump` 抓包定位问题。

掌握这一套方法后，大多数 Linux 网络故障都能在较短时间内定位到具体原因。如果是生产环境，建议每一步都记录观察结果，这样既便于复盘，也能提高后续类似问题的排查效率。