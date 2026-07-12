---
title: Linux host 命令
section: IT
category: Linux
---

# Linux host 命令

<img src="images/Linux.svg" width="300">

`host` 是 Linux 中一个用于 **DNS 查询** 的命令，可以用来查询域名、IP 地址以及各种 DNS 记录。它比 `nslookup` 更简洁，也常用于网络故障排查和脚本中。

## 基本语法

```bash
host [选项] 域名或IP地址 [DNS服务器]
```

其中：

* **域名或 IP 地址**：要查询的对象。
* **DNS服务器**（可选）：指定使用哪个 DNS 服务器进行查询。

---

## 常见用法

### 1. 查询域名对应的 IP 地址

```bash
host www.baidu.com
```

输出示例：

```text
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 110.242.68.66
www.a.shifen.com has address 39.156.66.18
```

说明：

* `is an alias` 表示 CNAME。
* `has address` 表示 IPv4 地址（A 记录）。

---

### 2. 查询 IP 对应的域名（反向解析）

```bash
host 8.8.8.8
```

输出：

```text
8.8.8.8.in-addr.arpa domain name pointer dns.google.
```

---

### 3. 查询 MX（邮件）记录

```bash
host -t mx gmail.com
```

输出：

```text
gmail.com mail is handled by 5 gmail-smtp-in.l.google.com.
```

---

### 4. 查询 NS（DNS服务器）记录

```bash
host -t ns example.com
```

---

### 5. 查询 TXT 记录

```bash
host -t txt example.com
```

常用于查看：

* SPF
* DKIM
* 域名验证信息

---

### 6. 查询 AAAA（IPv6）记录

```bash
host -t aaaa google.com
```

---

### 7. 查询 SOA 记录

```bash
host -t soa example.com
```

SOA（Start of Authority）记录包含：

* 主 DNS
* 管理员邮箱
* Serial（序列号）
* Refresh
* Retry
* Expire

---

### 8. 指定 DNS 服务器查询

例如使用 Google DNS：

```bash
host www.example.com 8.8.8.8
```

或者：

```bash
host www.example.com 1.1.1.1
```

---

## 常用参数

| 参数      | 说明                                  |
| ------- | ----------------------------------- |
| `-t`    | 指定查询记录类型                            |
| `-a`    | 查询所有记录（类似 `dig ANY`，部分 DNS 服务器可能限制） |
| `-v`    | 输出详细信息（Verbose）                     |
| `-W 秒数` | 设置查询超时时间                            |
| `-R 次数` | 设置重试次数                              |
| `-4`    | 仅使用 IPv4                            |
| `-6`    | 仅使用 IPv6                            |

例如：

```bash
host -v www.google.com
```

---

## 常见 DNS 记录类型

| 类型    | 含义               |
| ----- | ---------------- |
| A     | IPv4 地址          |
| AAAA  | IPv6 地址          |
| CNAME | 域名别名             |
| MX    | 邮件服务器            |
| NS    | 权威 DNS 服务器       |
| TXT   | 文本记录（SPF、DKIM 等） |
| SOA   | 域名授权信息           |
| PTR   | 反向解析             |
| SRV   | 服务定位记录           |

---

## 与 `dig`、`nslookup` 的区别

| 命令         | 特点          | 适用场景            |
| ---------- | ----------- | --------------- |
| `host`     | 简洁、易读       | 日常 DNS 查询、快速排查  |
| `dig`      | 功能最全面、输出详细  | 网络运维、DNS 故障分析   |
| `nslookup` | 历史悠久，交互模式方便 | 兼容旧系统，但已不推荐作为首选 |

通常建议：

* **快速查询**：使用 `host`
* **深入分析**：使用 `dig`
* **兼容旧环境**：使用 `nslookup`

---

## 常见排查示例

### 检查域名是否能解析

```bash
host www.example.com
```

### 查看邮件服务器

```bash
host -t mx example.com
```

### 查看域名使用的 DNS 服务器

```bash
host -t ns example.com
```

### 检查反向解析

```bash
host 192.168.1.10
```

如果返回：

```text
Host 10.1.168.192.in-addr.arpa not found: 3(NXDOMAIN)
```

表示该 IP 没有配置 PTR（反向解析）记录。

---

## 安装方法

如果系统没有 `host` 命令，可以安装 DNS 工具包：

**Debian/Ubuntu：**

```bash
sudo apt install dnsutils
```

**RHEL/CentOS/Rocky/AlmaLinux：**

```bash
sudo dnf install bind-utils
```

或旧版本：

```bash
sudo yum install bind-utils
```

---

## 总结

`host` 命令是 Linux 中轻量级的 DNS 查询工具，最适合快速查看域名解析结果。它支持查询 A、AAAA、MX、NS、TXT、SOA、PTR 等常见 DNS 记录，并可指定 DNS 服务器进行测试，是网络运维和故障排查中最常用的工具之一。