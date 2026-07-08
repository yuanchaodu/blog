---
title: Linux NetworkManager
section: IT
category: Linux
---

# Linux NetworkManager

<img src="images/Linux.svg" width="300">

# Linux NetworkManager 实用教程

NetworkManager 是 Linux 中常用的网络管理工具。它既能管理桌面电脑的有线网络和 Wi-Fi，也能管理服务器上的静态 IP、DNS、路由、VLAN、Bond、Bridge 和 VPN。官方将它定位为 Linux 的标准网络配置工具套件。([NetworkManager][1])

本文以命令行工具 `nmcli` 为主，适合 Linux 运维和服务器管理。官方资料可参考 [NetworkManager 官方文档](https://networkmanager.dev/) 和 [nmcli 官方示例](https://networkmanager.dev/docs/api/1.44.4/nmcli-examples.html)。

---

## 一、先理解 4 个核心概念

学习 NetworkManager，最重要的是分清下面四个东西。

### 1. NetworkManager

它是后台运行的网络管理服务，可以理解为“网络管理员”。

```bash
systemctl status NetworkManager
```

常用操作：

```bash
sudo systemctl start NetworkManager
sudo systemctl restart NetworkManager
sudo systemctl enable NetworkManager
```

NetworkManager 的目标是维护可用的网络连接，并根据配置管理 IP 地址、路由和其他网络参数。([wiki.debian.org][2])

---

### 2. Device：网络设备

Device 是实际或虚拟的网络接口，例如：

```text
ens160
enp1s0
wlan0
br0
bond0
```

查看设备：

```bash
nmcli device
```

也可以简写：

```bash
nmcli dev
```

典型输出：

```text
DEVICE   TYPE      STATE      CONNECTION
ens160   ethernet  connected  office
lo       loopback  connected  lo
```

这里：

```text
ens160  → 网卡
office  → 连接配置
```

这是学习 NetworkManager 最容易混淆的地方。

---

### 3. Connection：连接配置

Connection 不是网卡，而是一套“网络配置方案”。

例如，同一张网卡 `ens160` 可以准备两套配置：

```text
office-static
office-dhcp
```

第一套使用静态 IP，第二套使用 DHCP。

查看所有连接：

```bash
nmcli connection show
```

简写：

```bash
nmcli con show
```

NetworkManager 的核心就是“连接配置文件”。一个 Profile 保存一套网络参数，激活后再应用到设备上。([NetworkManager][3])

可以用一个简单比喻理解：

```text
Device      = 手机
Connection  = 手机中的 Wi-Fi 配置
```

一部手机可以保存很多 Wi-Fi 配置。同样，一张网卡也可以保存多个 Connection。

---

### 4. Active Connection：当前活动连接

已经启用的连接称为活动连接。

查看：

```bash
nmcli connection show --active
```

关系可以表示为：

```text
Connection Profile
        ↓ 激活
     Device
        ↓
IP + DNS + Route
```

---

# 二、nmcli 的基本结构

`nmcli` 是 NetworkManager 的命令行客户端，可以查看状态，也可以创建、修改、删除、启用和停用连接。([NetworkManager][4])

基本格式：

```bash
nmcli <对象> <操作>
```

最常见的对象：

```text
general      NetworkManager 总体状态
device       网络设备
connection   连接配置
```

常用简写：

```text
general      → g
device       → dev
connection   → con
```

例如：

```bash
nmcli device status
```

可以简写为：

```bash
nmcli dev status
```

---

# 三、查看当前网络状态

## 1. 查看总体状态

```bash
nmcli general status
```

查看主机名：

```bash
nmcli general hostname
```

---

## 2. 查看所有设备

```bash
nmcli device status
```

这是排查网络问题时建议执行的第一条命令。

---

## 3. 查看所有连接

```bash
nmcli connection show
```

查看当前活动连接：

```bash
nmcli connection show --active
```

---

## 4. 查看某个连接的详细配置

假设连接名称为：

```text
office
```

执行：

```bash
nmcli connection show office
```

只看 IPv4：

```bash
nmcli -f ipv4 connection show office
```

---

## 5. 查看设备实际使用的 IP

```bash
ip address
```

或者：

```bash
ip -br address
```

建议记住这个区别：

```text
nmcli con show office
```

查看的是“配置”。

```text
ip address
```

查看的是“当前实际状态”。

配置正确，不代表它已经成功应用。因此排障时，两边都要看。

---

# 四、配置 DHCP

假设：

```text
连接名称：office
网卡名称：ens160
```

修改为 DHCP：

```bash
sudo nmcli connection modify office \
  ipv4.method auto
```

然后重新激活：

```bash
sudo nmcli connection up office
```

如果没有现成的 Connection，可以新建：

```bash
sudo nmcli connection add \
  type ethernet \
  con-name office-dhcp \
  ifname ens160 \
  ipv4.method auto
```

启动：

```bash
sudo nmcli connection up office-dhcp
```

---

# 五、配置静态 IP

假设网络参数如下：

```text
IP地址：192.168.10.100/24
网关：192.168.10.1
DNS：192.168.10.10、192.168.10.11
```

## 方法一：修改现有连接

```bash
sudo nmcli connection modify office \
  ipv4.method manual \
  ipv4.addresses 192.168.10.100/24 \
  ipv4.gateway 192.168.10.1 \
  ipv4.dns "192.168.10.10 192.168.10.11"
```

重新激活：

```bash
sudo nmcli connection up office
```

---

## 方法二：新建静态 IP 连接

```bash
sudo nmcli connection add \
  type ethernet \
  con-name office-static \
  ifname ens160 \
  ipv4.method manual \
  ipv4.addresses 192.168.10.100/24 \
  ipv4.gateway 192.168.10.1 \
  ipv4.dns "192.168.10.10 192.168.10.11"
```

启动：

```bash
sudo nmcli connection up office-static
```

这种方式更适合服务器，因为可以保留原来的配置，方便切换和回退。([红帽文档][5])

---

# 六、修改 DNS

设置 DNS：

```bash
sudo nmcli con mod office \
  ipv4.dns "192.168.10.10 192.168.10.11"
```

如果不希望接受 DHCP 下发的 DNS：

```bash
sudo nmcli con mod office \
  ipv4.ignore-auto-dns yes
```

应用：

```bash
sudo nmcli con up office
```

检查：

```bash
nmcli dev show ens160
```

可以过滤 DNS：

```bash
nmcli dev show ens160 | grep DNS
```

NetworkManager 还可以通过 DNS Priority 控制多个连接之间的 DNS 优先级。([红帽文档][6])

---

# 七、启动、停止和删除连接

启动：

```bash
sudo nmcli connection up office
```

停止：

```bash
sudo nmcli connection down office
```

删除：

```bash
sudo nmcli connection delete office
```

注意：

```text
connection down
```

只是停用配置。

```text
connection delete
```

会删除配置。

生产服务器上执行 `delete` 前，应先确认有其他可用配置。

---

# 八、修改连接名称

例如把：

```text
Wired connection 1
```

改成：

```text
office
```

执行：

```bash
sudo nmcli connection modify \
  "Wired connection 1" \
  connection.id office
```

服务器上建议使用有意义的名称，例如：

```text
mgmt-static
production-vlan100
backup-network
office-dhcp
```

这样比系统自动生成的名称更容易维护。

---

# 九、添加第二个 IP 地址

先查看当前地址：

```bash
nmcli con show office
```

追加 IP：

```bash
sudo nmcli con mod office \
  +ipv4.addresses 192.168.10.101/24
```

应用：

```bash
sudo nmcli con up office
```

删除指定 IP：

```bash
sudo nmcli con mod office \
  -ipv4.addresses 192.168.10.101/24
```

这里要注意：

```text
ipv4.addresses   → 覆盖原值
+ipv4.addresses  → 追加
-ipv4.addresses  → 删除
```

这个规则非常实用。

---

# 十、配置静态路由

添加路由：

```bash
sudo nmcli con mod office \
  +ipv4.routes "10.20.0.0/16 192.168.10.254"
```

应用：

```bash
sudo nmcli con up office
```

查看：

```bash
ip route
```

删除：

```bash
sudo nmcli con mod office \
  -ipv4.routes "10.20.0.0/16 192.168.10.254"
```

对于企业服务器，建议把永久路由放入 Connection，而不是只执行：

```bash
ip route add
```

因为后者通常只修改当前运行状态，重启后可能丢失。

---

# 十一、配置 VLAN

NetworkManager 支持 VLAN、Bond、Bridge、Tunnel 等高级网络结构。([红帽文档][7])

假设：

```text
物理网卡：ens160
VLAN ID：100
IP：192.168.100.10/24
```

创建 VLAN：

```bash
sudo nmcli con add \
  type vlan \
  con-name vlan100 \
  ifname ens160.100 \
  dev ens160 \
  id 100
```

配置 IP：

```bash
sudo nmcli con mod vlan100 \
  ipv4.method manual \
  ipv4.addresses 192.168.100.10/24 \
  ipv4.gateway 192.168.100.1
```

启动：

```bash
sudo nmcli con up vlan100
```

查看：

```bash
ip -d link show ens160.100
```

---

# 十二、Wi-Fi 基本操作

打开 Wi-Fi：

```bash
nmcli radio wifi on
```

扫描：

```bash
nmcli device wifi list
```

连接：

```bash
nmcli device wifi connect "SSID名称"
```

如果系统需要密码，建议使用交互方式输入，避免密码直接出现在 Shell 历史记录中。

断开：

```bash
nmcli device disconnect wlan0
```

重新连接：

```bash
nmcli device connect wlan0
```

---

# 十三、使用 nmtui

如果觉得 `nmcli` 命令太长，可以使用：

```bash
nmtui
```

它提供文本菜单，可以完成：

```text
修改连接
激活连接
修改主机名
```

NetworkManager 支持命令行、文本界面和其他管理方式；在 RHEL 的高级网络配置中，`nmcli` 和 `nmtui` 都是正式支持的工具。([红帽文档][8])

对于初学者，可以这样分工：

```text
日常手工配置  → nmtui
批量操作      → nmcli
自动化脚本    → nmcli
故障排查      → nmcli + ip
```

---

# 十四、配置文件在哪里

NetworkManager 的原生格式是 keyfile。官方文档指出，一个连接配置由多个 Setting 组成，例如：

```text
connection
ipv4
ipv6
ethernet
```

每个 Setting 再包含具体属性。([NetworkManager][9])

常见配置目录：

```text
/etc/NetworkManager/system-connections/
```

查看：

```bash
sudo ls -l /etc/NetworkManager/system-connections/
```

一般建议优先使用：

```bash
nmcli
```

而不是直接编辑文件。

原因很简单：`nmcli` 会帮助检查参数，并通知 NetworkManager 更新配置。

---

# 十五、Ubuntu 要特别注意 Netplan

在 Ubuntu 上，特别是服务器系统中，还要理解 Netplan。

Ubuntu Server 通常使用 Netplan 描述网络配置，而 Netplan 可以选择：

```text
systemd-networkd
NetworkManager
```

作为后端。([Ubuntu][10])

例如：

```yaml
network:
  version: 2
  renderer: NetworkManager
```

这表示：

```text
Netplan
   ↓
NetworkManager
   ↓
网络设备
```

较新的 Netplan 文档仍明确支持 `NetworkManager` 作为 renderer；Ubuntu Desktop 默认场景也会把网络管理交给 NetworkManager。([Netplan][11])

因此，在 Ubuntu 上修改网络前，先执行：

```bash
ls /etc/netplan/
```

再检查：

```bash
networkctl
```

以及：

```bash
nmcli device
```

先判断到底是谁在管理网卡，不要让 Netplan、NetworkManager 和手工配置互相“打架”。

---

# 十六、故障排查的标准流程

网络故障时，建议按照固定顺序检查：

1. NetworkManager 服务是否正常：

```bash
systemctl status NetworkManager
```

2. 网卡是否被识别：

```bash
nmcli device status
```

3. Connection 是否存在：

```bash
nmcli connection show
```

4. Connection 是否激活：

```bash
nmcli connection show --active
```

5. IP 是否正确：

```bash
ip -br address
```

6. 路由是否正确：

```bash
ip route
```

7. DNS 是否正确：

```bash
nmcli device show
```

8. 查看日志：

```bash
journalctl -u NetworkManager
```

查看本次启动：

```bash
journalctl -u NetworkManager -b
```

实时跟踪：

```bash
journalctl -u NetworkManager -f
```

NetworkManager 的主要日志进入 systemd journal，也可以提高日志级别进行深入排查。([红帽文档][12])

---

# 十七、生产服务器最重要的操作习惯

远程修改服务器网络时，最大的风险不是命令写错，而是把自己的 SSH 连接切断。

建议遵守这几个原则：

* 修改前先执行 `nmcli con show`，确认 Connection 名称。
* 先新建配置，再切换，不要直接删除旧配置。
* 修改 IP、网关和默认路由时，要准备控制台或其他管理通道。
* 操作后同时检查 `nmcli`、`ip address` 和 `ip route`。
* 重要服务器要记录修改前后的网络参数。

尤其不要在不确定的情况下直接执行：

```bash
nmcli networking off
```

远程服务器上，这条命令可能立即导致网络中断。

---

# 十八、建议记住的 12 条命令

```bash
# 1. 查看设备
nmcli dev status

# 2. 查看所有连接
nmcli con show

# 3. 查看活动连接
nmcli con show --active

# 4. 查看连接详情
nmcli con show <连接名>

# 5. 启动连接
sudo nmcli con up <连接名>

# 6. 停止连接
sudo nmcli con down <连接名>

# 7. 查看 IP
ip -br address

# 8. 查看路由
ip route

# 9. 查看设备详细信息
nmcli dev show

# 10. 查看服务
systemctl status NetworkManager

# 11. 查看日志
journalctl -u NetworkManager -b

# 12. 打开文本配置界面
nmtui
```

NetworkManager 真正的核心并不是记住大量命令，而是理解一句话：

```text
NetworkManager 通过 Connection Profile 管理 Device。
```

理解这个模型后，静态 IP、DNS、路由、VLAN 和 Bond 都只是给不同类型的 Connection 增加属性。

[1]: https://networkmanager.dev/ "NetworkManager"
[2]: https://wiki.debian.org/NetworkManager "NetworkManager"
[3]: https://networkmanager.dev/docs/api/latest/nm-settings-nmcli.html "nm-settings-nmcli: NetworkManager Reference Manual"
[4]: https://networkmanager.dev/docs/api/1.44.4/nmcli-examples.html "nmcli-examples: NetworkManager Reference Manual"
[5]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html-single/configuring_and_managing_networking/index "Configuring and managing networking"
[6]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/configuring-the-order-of-dns-servers_configuring-and-managing-networking "Chapter 25. Configuring the order of DNS servers"
[7]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/index "Configuring and managing networking"
[8]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/configuring-network-bonding_configuring-and-managing-networking "Chapter 3. Configuring a network bond"
[9]: https://networkmanager.dev/docs/man-pages/ "man pages - NetworkManager"
[10]: https://ubuntu.com/server/docs/explanation/networking/configuring-networks/ "Configuring networks - Ubuntu Server documentation"
[11]: https://netplan.readthedocs.io/en/latest/nm-all/ "NetworkManager default configuration - Netplan documentation"
[12]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/introduction-to-networkmanager-debugging_configuring-and-managing-networking "Chapter 33. Introduction to NetworkManager Debugging"