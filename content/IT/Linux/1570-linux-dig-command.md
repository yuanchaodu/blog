---
title: Linux dig 命令
section: IT
category: Linux
---

# Linux dig 命令

<img src="images/Linux.svg" width="300">

`dig`（**Domain Information Groper**）是 Linux 中最常用的 DNS 查询工具之一，用于查询域名解析信息、排查 DNS 故障、验证 DNS 配置等。相比 `nslookup` 和 `host`，`dig` 输出更详细，因此也是网络管理员最常用的 DNS 调试工具。

---

# 一、基本语法

```bash
dig [@DNS服务器] 域名 [查询类型]
```

例如：

```bash
dig www.baidu.com
```

---

# 二、最常用的命令

## 1、查询域名 A 记录（默认）

```bash
dig www.baidu.com
```

输出类似：

```text
;; ANSWER SECTION:
www.baidu.com. 600 IN A 39.156.66.14
www.baidu.com. 600 IN A 110.242.68.4
```

表示：

* TTL：600 秒
* 类型：A
* IP 地址：

  * 39.156.66.14
  * 110.242.68.4

---

## 2、指定 DNS 服务器查询

例如使用 Google DNS：

```bash
dig @8.8.8.8 www.baidu.com
```

使用 Cloudflare：

```bash
dig @1.1.1.1 www.baidu.com
```

使用企业 DNS：

```bash
dig @192.168.1.1 www.example.com
```

常用于比较不同 DNS 的解析结果是否一致。

---

## 3、查询 MX（邮件服务器）

```bash
dig example.com MX
```

输出：

```text
example.com. 3600 IN MX 10 mail.example.com.
```

表示邮件服务器：

```
mail.example.com
```

---

## 4、查询 NS（DNS服务器）

```bash
dig example.com NS
```

结果：

```text
example.com. IN NS ns1.example.com.
example.com. IN NS ns2.example.com.
```

---

## 5、查询 TXT 记录

```bash
dig example.com TXT
```

可查看：

* SPF
* DKIM
* 域名验证信息
* Google 验证

例如：

```text
"v=spf1 include:_spf.google.com ~all"
```

---

## 6、查询 CNAME

```bash
dig www.example.com CNAME
```

输出：

```text
www.example.com. IN CNAME example.com.
```

---

## 7、查询 AAAA（IPv6）

```bash
dig google.com AAAA
```

---

## 8、查询 SOA

```bash
dig example.com SOA
```

输出：

```text
example.com. IN SOA
ns1.example.com.
admin.example.com.
2025070101
7200
3600
1209600
3600
```

SOA 中包含：

* 主 DNS
* 管理员邮箱
* Serial（版本号）
* Refresh
* Retry
* Expire
* Minimum TTL

---

# 三、简洁输出

默认输出很多调试信息。

如果只需要 IP：

```bash
dig +short www.baidu.com
```

输出：

```text
39.156.66.14
110.242.68.4
```

这是最常用的参数。

---

只查看 MX：

```bash
dig +short example.com MX
```

输出：

```text
10 mail.example.com.
```

---

# 四、查看完整解析过程

## Trace（递归查询）

```bash
dig +trace www.baidu.com
```

它会依次查询：

```
Root DNS
↓

.com
↓

baidu.com
↓

www.baidu.com
```

适用于：

* DNS 故障分析
* 权威 DNS 是否正常
* DNS 委派是否正确

---

# 五、查看 DNSSEC

```bash
dig +dnssec example.com
```

会返回：

```
RRSIG
DNSKEY
DS
```

用于验证 DNSSEC 配置。

---

# 六、反向解析 IP

例如：

```bash
dig -x 8.8.8.8
```

输出：

```text
8.8.8.8.in-addr.arpa.

PTR dns.google.
```

相当于：

```bash
nslookup 8.8.8.8
```

---

# 七、查看所有记录（ANY）

```bash
dig example.com ANY
```

需要注意的是，现代 DNS 服务器通常会限制 `ANY` 查询，因此可能不会返回所有记录。

---

# 八、查看 TTL

例如：

```bash
dig www.example.com
```

结果：

```text
www.example.com. 300 IN A 192.168.1.10
```

其中：

```
300
```

表示 TTL 为 300 秒。

---

# 九、批量查询

创建文件：

```
domains.txt
```

内容：

```text
google.com
baidu.com
openai.com
```

使用 Shell 批量查询：

```bash
while read domain; do
    dig +short "$domain"
done < domains.txt
```

---

# 十、统计查询时间

输出中可以看到：

```text
;; Query time: 18 msec
```

表示 DNS 查询耗时。

---

# 十一、查看使用的 DNS

如果没有指定 DNS：

```bash
dig www.baidu.com
```

默认使用：

```
/etc/resolv.conf
```

查看配置：

```bash
cat /etc/resolv.conf
```

例如：

```text
nameserver 192.168.1.1
```

---

# 十二、输出结果说明

典型输出：

```text
; <<>> DiG 9.18 <<>> www.example.com

;; QUESTION SECTION:
;www.example.com.      IN A

;; ANSWER SECTION:
www.example.com. 300 IN A 192.168.1.10

;; AUTHORITY SECTION:
example.com. 172800 IN NS ns1.example.com.

;; ADDITIONAL SECTION:
ns1.example.com. 300 IN A 192.168.1.2

;; Query time: 16 msec
;; SERVER: 192.168.1.1#53
;; WHEN: Sun Jul 12 16:20:00 CST 2026
```

各部分含义如下：

| 部分                 | 说明                   |
| ------------------ | -------------------- |
| QUESTION SECTION   | 查询请求的内容              |
| ANSWER SECTION     | DNS 返回的最终解析结果        |
| AUTHORITY SECTION  | 返回负责该域名的权威 DNS 服务器信息 |
| ADDITIONAL SECTION | 附加信息，如权威 DNS 的 IP 地址 |
| Query time         | 查询耗时                 |
| SERVER             | 实际响应查询的 DNS 服务器      |
| WHEN               | 查询时间                 |

---

# 十三、安装 dig

`dig` 工具通常包含在 DNS 工具包中，不同发行版的软件包名称不同：

* **Ubuntu / Debian**

  ```bash
  sudo apt install dnsutils
  ```

* **CentOS 7**

  ```bash
  sudo yum install bind-utils
  ```

* **CentOS Stream / Rocky Linux / AlmaLinux / RHEL 8+**

  ```bash
  sudo dnf install bind-utils
  ```

* **openSUSE**

  ```bash
  sudo zypper install bind-utils
  ```

---

## 常用命令速查

| 功能            | 命令                                              |
| ------------- | ----------------------------------------------- |
| 查询 A 记录       | `dig example.com`                               |
| 查询 A 记录（简洁输出） | `dig +short example.com`                        |
| 指定 DNS 服务器    | `dig @8.8.8.8 example.com`                      |
| 查询 MX 记录      | `dig example.com MX`                            |
| 查询 NS 记录      | `dig example.com NS`                            |
| 查询 TXT 记录     | `dig example.com TXT`                           |
| 查询 CNAME      | `dig www.example.com CNAME`                     |
| 查询 AAAA（IPv6） | `dig example.com AAAA`                          |
| 查询 SOA        | `dig example.com SOA`                           |
| 反向解析 IP       | `dig -x 8.8.8.8`                                |
| 查看递归解析过程      | `dig +trace example.com`                        |
| 查看 DNSSEC 信息  | `dig +dnssec example.com`                       |
| 查看 TTL        | `dig example.com`（查看 `ANSWER SECTION` 中 TTL 字段） |

对于日常运维和故障排查，最常用的几条命令是：

```bash
# 查看域名解析结果
dig +short example.com

# 指定 DNS 服务器查询
dig @8.8.8.8 example.com

# 查看完整解析详情
dig example.com

# 跟踪 DNS 递归解析路径
dig +trace example.com

# 反向解析 IP
dig -x 8.8.8.8
```

掌握这几种用法，基本可以满足大多数 DNS 查询和故障诊断需求。