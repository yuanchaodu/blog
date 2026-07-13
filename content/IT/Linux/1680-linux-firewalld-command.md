---
title: Linux Firewalld 防火墙
section: IT
---

# Linux Firewalld 防火墙

<img src="images/Linux.svg" width="300">

Firewalld 是 Linux 上广泛使用的**动态防火墙管理工具**，主要用于管理 Linux 内核的 Netfilter（目前大多数发行版底层使用 **nftables**，也兼容 iptables）。它提供了比直接编写 nftables/iptables 规则更简单、更安全的管理方式。Firewalld 的核心特点是**动态修改规则无需中断现有连接**，并将配置分为**运行时（Runtime）**和**永久（Permanent）**两套配置。([firewalld][1])

## 一、Firewalld 的主要特点

* 动态修改防火墙规则，无需重启服务
* 基于**Zone（区域）**管理不同网络的信任级别
* 支持 IPv4、IPv6
* 支持服务（Service）、端口（Port）、协议（Protocol）、源地址（Source）等多种管理方式
* 提供命令行（firewall-cmd）、图形界面（firewall-config）和 D-Bus API 接口([firewalld][2])

---

## 二、Firewalld 架构

```
                用户
                  │
          firewall-cmd
                  │
             firewalld
                  │
           nftables（默认）
        （或 iptables 后端）
                  │
           Linux Kernel
```

目前 RHEL、CentOS、Rocky Linux、AlmaLinux、Fedora 等发行版默认都使用 Firewalld。([firewalld][2])

---

## 三、运行状态管理

### 查看状态

```bash
systemctl status firewalld
```

或者

```bash
firewall-cmd --state
```

输出：

```
running
```

---

### 启动

```bash
systemctl start firewalld
```

---

### 停止

```bash
systemctl stop firewalld
```

---

### 开机启动

```bash
systemctl enable firewalld
```

---

### 开机禁用

```bash
systemctl disable firewalld
```

---

## 四、Runtime 与 Permanent

Firewalld 最大特点就是两套配置。

### Runtime（运行时）

立即生效

```
firewall-cmd --add-service=http
```

重启以后失效。

---

### Permanent（永久）

```
firewall-cmd --permanent --add-service=http
```

不会立即生效，需要重新加载：

```
firewall-cmd --reload
```

---

通常建议：

```
修改永久配置
        ↓
reload
```

---

## 五、Zone（区域）

Zone 是 Firewalld 最重要的概念。

每个网卡都属于某个 Zone。

常见 Zone：

| Zone     | 信任等级  | 适用场景   |
| -------- | ----- | ------ |
| drop     | 最低    | 丢弃所有连接 |
| block    | 很低    | 拒绝所有连接 |
| public   | 默认    | 公网服务器  |
| external | 外网    |        |
| dmz      | DMZ 区 |        |
| work     | 办公网   |        |
| home     | 家庭网络  |        |
| internal | 内网    |        |
| trusted  | 全部允许  |        |

查看所有 Zone：

```bash
firewall-cmd --get-zones
```

查看当前默认 Zone：

```bash
firewall-cmd --get-default-zone
```

查看活动 Zone：

```bash
firewall-cmd --get-active-zones
```

例如：

```
public
  interfaces: ens192
```

---

## 六、查看当前规则

查看默认 Zone：

```bash
firewall-cmd --list-all
```

查看指定 Zone：

```bash
firewall-cmd --zone=public --list-all
```

输出类似：

```
public
  target: default
  interfaces: ens33
  services: ssh dhcpv6-client
  ports:
```

---

## 七、开放服务

例如开放 HTTP：

```
firewall-cmd --add-service=http
```

永久开放：

```
firewall-cmd --permanent --add-service=http
```

查看支持哪些服务：

```
firewall-cmd --get-services
```

常见服务：

```
http
https
ssh
ftp
dns
smtp
mysql
samba
```

---

## 八、开放端口

开放 TCP 8080：

```
firewall-cmd --add-port=8080/tcp
```

永久开放：

```
firewall-cmd --permanent --add-port=8080/tcp
```

UDP：

```
firewall-cmd --add-port=53/udp
```

---

## 九、删除规则

删除端口：

```
firewall-cmd --remove-port=8080/tcp
```

永久删除：

```
firewall-cmd --permanent --remove-port=8080/tcp
```

删除服务：

```
firewall-cmd --remove-service=http
```

---

## 十、重新加载配置

```
firewall-cmd --reload
```

不会中断已有连接，是 Firewalld 的优势之一。([firewalld][3])

---

## 十一、常用命令汇总

| 功能        | 命令                                |
| --------- | --------------------------------- |
| 查看状态      | `firewall-cmd --state`            |
| 查看默认 Zone | `firewall-cmd --get-default-zone` |
| 查看活动 Zone | `firewall-cmd --get-active-zones` |
| 查看规则      | `firewall-cmd --list-all`         |
| 查看所有 Zone | `firewall-cmd --get-zones`        |
| 查看所有服务    | `firewall-cmd --get-services`     |
| 开放端口      | `--add-port=80/tcp`               |
| 删除端口      | `--remove-port=80/tcp`            |
| 开放服务      | `--add-service=http`              |
| 删除服务      | `--remove-service=http`           |
| 重新加载      | `firewall-cmd --reload`           |

---

## 十二、典型应用示例

### Web 服务器

开放 HTTP 和 HTTPS：

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

---

### SSH 服务器

开放 SSH：

```bash
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload
```

---

### MySQL 数据库

开放 3306：

```bash
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload
```

---

### 查看开放端口

```
firewall-cmd --list-ports
```

---

## 十三、配置文件位置

Firewalld 的主要配置目录包括：

* `/etc/firewalld/`：系统管理员自定义配置。
* `/usr/lib/firewalld/`：软件包提供的默认配置，不建议直接修改。([firewalld][1])

---

## 十四、Firewalld 与 iptables/nftables 的关系

| 项目      | Firewalld | nftables | iptables |
| ------- | --------- | -------- | -------- |
| 定位      | 防火墙管理工具   | 内核防火墙框架  | 旧版防火墙框架  |
| 易用性     | 高         | 中        | 较低       |
| 动态修改    | 支持        | 支持       | 支持但管理复杂  |
| Zone 管理 | 支持        | 不直接支持    | 不支持      |
| 当前推荐    | ★★★★★     | ★★★★★    | 兼容维护     |

现代 Linux（如 RHEL 8/9、Rocky Linux 8/9、AlmaLinux 8/9、Fedora 等）默认采用 **nftables** 作为底层实现，而 Firewalld 负责提供统一、易用的管理接口。([firewalld][1])

[1]: https://firewalld.org/documentation/man-pages/firewalld "Documentation - Manual Pages - firewalld | firewalld"
[2]: https://firewalld.org/?lang=en "Home | firewalld"
[3]: https://firewalld.org/documentation/index.html "Documentation | firewalld"