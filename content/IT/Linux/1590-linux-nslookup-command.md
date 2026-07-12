---
title: Linux nslookup 命令
section: IT
category: Linux
---

# Linux nslookup 命令

<img src="images/Linux.svg" width="300">

`nslookup` 是 Linux（以及 Windows）中用于**查询 DNS（域名系统）**信息的命令，可以用来查看域名对应的 IP 地址、查询 MX 记录、NS 记录、TXT 记录等，也是排查 DNS 故障最常用的工具之一。

## 一、基本语法

```bash
nslookup [选项] 域名 [DNS服务器]
```

例如：

```bash
nslookup www.baidu.com
```

输出示例：

```text
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   www.baidu.com
Address: 110.242.68.66
```

含义：

* **Server**：当前使用的 DNS 服务器
* **Address**：DNS 服务器地址
* **Non-authoritative answer**：非权威服务器返回的结果（来自缓存）
* **Name**：查询的域名
* **Address**：解析出的 IP 地址

---

## 二、常见用法

### 1. 查询域名对应 IP

```bash
nslookup www.google.com
```

返回：

```text
Name:    www.google.com
Address: 142.250.72.196
```

---

### 2. 反向解析 IP

查询 IP 对应哪个域名：

```bash
nslookup 8.8.8.8
```

结果：

```text
8.8.8.8.in-addr.arpa
name = dns.google.
```

---

### 3. 指定 DNS 服务器查询

例如使用 Google DNS：

```bash
nslookup www.baidu.com 8.8.8.8
```

或者使用 Cloudflare DNS：

```bash
nslookup www.baidu.com 1.1.1.1
```

常用于比较不同 DNS 的解析结果。

---

### 4. 查询 MX（邮件）记录

进入交互模式：

```bash
nslookup
```

然后：

```text
> set type=mx
> gmail.com
```

输出类似：

```text
gmail.com mail exchanger = 10 smtp.google.com
```

---

### 5. 查询 NS（DNS服务器）记录

```text
> set type=ns
> baidu.com
```

输出：

```text
baidu.com
nameserver = dns.baidu.com
```

---

### 6. 查询 TXT 记录

```text
> set type=txt
> google.com
```

可以看到：

* SPF
* 域名验证
* DKIM 等 TXT 信息

---

### 7. 查询 SOA 记录

```text
> set type=soa
> example.com
```

返回域名授权信息：

* 主 DNS
* 管理员邮箱
* Serial
* Refresh
* Retry
* Expire

---

### 8. 查询所有记录（部分 DNS 已不支持）

```text
> set type=any
> example.com
```

很多公共 DNS（如 Google Public DNS）出于安全和性能原因，已经限制或不再返回 `ANY` 查询结果，因此可能得不到完整信息。

---

## 三、交互模式

直接输入：

```bash
nslookup
```

进入：

```text
>
```

可以连续查询：

```text
> server 8.8.8.8
> www.baidu.com
> set type=mx
> baidu.com
> set type=ns
> google.com
> exit
```

常用命令：

| 命令             | 作用         |
| -------------- | ---------- |
| server 8.8.8.8 | 切换 DNS 服务器 |
| set type=a     | 查询 A 记录    |
| set type=aaaa  | 查询 IPv6    |
| set type=mx    | 邮件服务器      |
| set type=ns    | 名称服务器      |
| set type=txt   | TXT        |
| set type=soa   | SOA        |
| exit           | 退出         |

---

## 四、常见记录类型

| 类型    | 说明               |
| ----- | ---------------- |
| A     | IPv4 地址          |
| AAAA  | IPv6 地址          |
| MX    | 邮件服务器            |
| NS    | DNS 权威服务器        |
| TXT   | 文本记录（SPF、DKIM 等） |
| CNAME | 别名记录             |
| SOA   | 域名授权信息           |
| PTR   | 反向解析记录           |
| SRV   | 服务定位记录           |

---

## 五、常见错误

### NXDOMAIN

```text
** server can't find abc.example.com: NXDOMAIN
```

表示：

> 域名不存在。

---

### SERVFAIL

```text
** server can't find example.com: SERVFAIL
```

表示：

* DNS 服务器故障
* DNSSEC 验证失败
* 上游 DNS 无法响应

---

### Timed out

```text
DNS request timed out.
```

通常表示：

* DNS 不可达
* 防火墙阻止 UDP/TCP 53 端口
* 网络故障

---

## 六、常见排障示例

### 判断本机 DNS 是否正常

```bash
nslookup www.baidu.com
```

如果能够返回 IP，说明本机到 DNS 的解析正常。

---

### 判断指定 DNS 是否正常

```bash
nslookup www.baidu.com 114.114.114.114
```

如果指定 DNS 可以解析，而默认 DNS 不行，则问题通常出在默认 DNS 配置。

---

### 比较不同 DNS 的解析结果

```bash
nslookup www.example.com 8.8.8.8
nslookup www.example.com 1.1.1.1
nslookup www.example.com 223.5.5.5
```

可以检查是否存在解析差异或 DNS 污染。

---

## 七、与 dig 的比较

| 功能          | nslookup | dig |
| ----------- | -------- | --- |
| 使用简单        | ✔        | 一般  |
| 输出简洁        | ✔        | 较详细 |
| 脚本友好        | 一般       | ✔   |
| 支持更多 DNS 参数 | 较少       | ✔   |
| DNSSEC 调试   | 较弱       | ✔   |
| 现代 Linux 推荐 | 一般       | ✔   |

目前在大多数 Linux 发行版中，更推荐使用 `dig`（通常由 `bind-utils` 或 `dnsutils` 软件包提供）进行 DNS 查询，因为它输出更完整、便于脚本处理，并支持 DNSSEC、EDNS 等高级功能；`nslookup` 更适合快速、简单的 DNS 查询。