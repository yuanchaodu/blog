---
title: Linux nmcli 命令
section: IT
category: Linux
---

# Linux nmcli 命令

<img src="images/Linux.svg" width="300">

`nmcli` 是 Linux 中管理网络的命令行工具，属于 **NetworkManager**。常用于查看、配置、启用、关闭网络连接。

## 常用命令

查看网络设备：

```bash
nmcli device status
```

查看网络连接：

```bash
nmcli connection show
```

查看当前 IP：

```bash
ip addr
```

或：

```bash
nmcli device show
```

## 启用/关闭网络连接

启用连接：

```bash
nmcli connection up 连接名
```

关闭连接：

```bash
nmcli connection down 连接名
```

例如：

```bash
nmcli connection up ens33
```

## 配置静态 IP

```bash
nmcli connection modify ens33 \
ipv4.addresses 192.168.1.100/24 \
ipv4.gateway 192.168.1.1 \
ipv4.dns "8.8.8.8 114.114.114.114" \
ipv4.method manual
```

然后重启连接：

```bash
nmcli connection down ens33
nmcli connection up ens33
```

## 改为 DHCP 自动获取 IP

```bash
nmcli connection modify ens33 ipv4.method auto
nmcli connection down ens33
nmcli connection up ens33
```

## 查看 Wi-Fi

```bash
nmcli device wifi list
```

连接 Wi-Fi：

```bash
nmcli device wifi connect "WiFi名称" password "密码"
```

## 常见排查

查看网络是否由 NetworkManager 管理：

```bash
nmcli general status
```

重启 NetworkManager：

```bash
systemctl restart NetworkManager
```

查看详细连接信息：

```bash
nmcli connection show ens33
```

常用套路是：

```bash
nmcli device status
nmcli connection show
nmcli connection modify ...
nmcli connection up ...
```