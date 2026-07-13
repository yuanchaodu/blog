---
title: Linux nftables 基础
section: IT
category: Linux
---

# Linux nftables 基础

<img src="images/Linux.svg" width="300">

Linux **nftables** 是 Linux 新一代的防火墙框架，用来替代 **iptables、ip6tables、arptables、ebtables** 等多个工具。从 Linux 3.13 开始进入内核，目前几乎所有主流 Linux 发行版（如 Debian、Ubuntu、RHEL、Rocky、AlmaLinux、Fedora 等）都默认支持。

如果你已经接触过 iptables，可以把 nftables 理解为**统一、简洁、性能更高的新一代包过滤框架**。

---

# 一、什么是 nftables

### 通俗理解

可以把 Linux 网络想象成一个工厂的大门。

所有网络数据包都是车辆。

nftables 就是门卫。

门卫可以根据很多条件决定：

* 放行（accept）
* 拒绝（drop）
* 丢弃（reject）
* 修改地址（NAT）
* 记录日志（log）
* 限速（limit）
* 标记数据包（mark）

整个流程如下：

```
互联网
     │
     ▼
+--------------+
|   网卡       |
+--------------+
      │
      ▼
+----------------+
|   nftables     |
|                |
| 规则1          |
| 规则2          |
| 规则3          |
+----------------+
      │
      ▼
Linux 系统
```

---

# 二、为什么要替代 iptables

iptables 存在很多历史问题：

例如：

```
iptables

filter
    INPUT
    OUTPUT
    FORWARD

nat
    PREROUTING
    POSTROUTING

mangle

raw
```

不同功能分散到多个 table。

而 nftables：

* IPv4 和 IPv6 共用
* NAT 共用
* bridge 共用
* inet 家族统一
* 支持集合(Set)
* 支持映射(Map)
* 支持变量
* 规则可以原子更新
* 性能更高

因此：

```
iptables
    ↓

多个命令
多个模块
多个规则链

====================

nftables

一个命令
一个规则集
统一管理
```

---

# 三、nftables 的核心概念

理解 nftables，只需要掌握几个对象即可。

```
Ruleset
│
├── Table
│      │
│      ├── Chain
│      │      │
│      │      ├── Rule
│      │      ├── Rule
│      │      └── Rule
│      │
│      └── Chain
│
└── Table
```

也就是说：

```
Ruleset
    └── Table
            └── Chain
                    └── Rule
```

这是 nftables 最重要的层级关系。

---

## 1）Ruleset（规则集）

整个系统所有规则组成一个 Ruleset。

查看：

```bash
sudo nft list ruleset
```

例如：

```
ruleset
├── table inet filter
├── table ip nat
└── table bridge filter
```

---

## 2）Table（表）

Table 用于分类规则。

例如：

```
table inet filter
table ip nat
table ip6 nat
```

常见类型：

| Family | 作用        |
| ------ | --------- |
| ip     | IPv4      |
| ip6    | IPv6      |
| inet   | IPv4+IPv6 |
| arp    | ARP       |
| bridge | 二层桥接      |
| netdev | 网卡入口      |

例如创建：

```bash
sudo nft add table inet filter
```

---

## 3）Chain（链）

Chain 相当于规则容器。

例如：

```
INPUT

OUTPUT

FORWARD
```

创建：

```bash
sudo nft add chain inet filter input \
'{ type filter hook input priority 0; }'
```

这里：

```
type filter

hook input

priority 0
```

表示：

* 过滤链
* 挂到 INPUT
* 优先级 0

---

## 4）Rule（规则）

真正工作的就是 Rule。

例如：

允许 SSH：

```bash
sudo nft add rule inet filter input tcp dport 22 accept
```

拒绝 ICMP：

```bash
sudo nft add rule inet filter input ip protocol icmp drop
```

允许 HTTP：

```bash
sudo nft add rule inet filter input tcp dport 80 accept
```

---

# 四、Hook（挂钩）

Hook 表示规则插入 Linux 网络协议栈的位置。

主要有：

```
        收包
          │
          ▼

    PREROUTING
          │
          ▼

     路由判断
      │      │
      │      │
      ▼      ▼

 INPUT    FORWARD
   │         │
   ▼         ▼

 本机      转发

   │
   ▼

 OUTPUT

   │
   ▼

POSTROUTING

   │
   ▼

发送
```

常见 Hook：

| Hook        | 用途   |
| ----------- | ---- |
| prerouting  | 路由前  |
| input       | 本机接收 |
| forward     | 转发   |
| output      | 本机发送 |
| postrouting | 发包前  |

---

# 五、基本命令

查看版本：

```bash
nft --version
```

查看全部规则：

```bash
sudo nft list ruleset
```

查看 table：

```bash
sudo nft list tables
```

创建 table：

```bash
sudo nft add table inet filter
```

删除 table：

```bash
sudo nft delete table inet filter
```

查看 chain：

```bash
sudo nft list table inet filter
```

添加规则：

```bash
sudo nft add rule inet filter input tcp dport 22 accept
```

删除规则：

先查看规则句柄（handle）：

```bash
sudo nft -a list chain inet filter input
```

输出示例：

```
tcp dport 22 accept # handle 5
```

删除：

```bash
sudo nft delete rule inet filter input handle 5
```

---

# 六、一个最简单的防火墙示例

假设目标是：

* 默认拒绝所有进入连接
* 允许已建立连接
* 允许本地回环接口
* 允许 SSH
* 允许 HTTP
* 允许 HTTPS

可以使用如下配置：

```nft
table inet filter {

    chain input {

        type filter hook input priority 0;

        policy drop;

        ct state established,related accept

        iif lo accept

        tcp dport 22 accept

        tcp dport 80 accept

        tcp dport 443 accept
    }

    chain forward {

        type filter hook forward priority 0;

        policy drop;
    }

    chain output {

        type filter hook output priority 0;

        policy accept;
    }
}
```

说明：

* `policy drop`：默认拒绝所有未匹配规则的入站流量。
* `ct state established,related accept`：允许已经建立或相关的连接返回流量。
* `iif lo accept`：允许本机回环接口（`lo`）通信。
* `output` 链默认放行，允许本机主动发起连接。

> **注意**：如果通过 SSH 远程管理服务器，建议先添加允许 SSH 的规则，再将 `input` 链策略设置为 `drop`，否则可能导致远程连接中断。

---

# 七、配置文件

配置文件通常为：

```
/etc/nftables.conf
```

加载配置：

```bash
sudo nft -f /etc/nftables.conf
```

保存当前规则：

```bash
sudo sh -c "nft list ruleset > /etc/nftables.conf"
```

开机启动（以使用 `systemd` 的发行版为例）：

```bash
sudo systemctl enable nftables
sudo systemctl start nftables
```

---

# 八、常用高级特性

相比 iptables，nftables 提供了更灵活的表达能力：

| 特性                        | 作用                           | 示例                      |
| ------------------------- | ---------------------------- | ----------------------- |
| Set（集合）                   | 一条规则匹配多个值                    | `tcp dport {22,80,443}` |
| Map（映射）                   | 根据键映射到不同结果                   | IP 地址映射到不同标记或动作         |
| Verdict Map               | 根据条件直接决定 `accept`、`drop` 等动作 | 简化复杂分支逻辑                |
| Connection Tracking（连接跟踪） | 按连接状态过滤                      | `ct state established`  |
| Counter（计数器）              | 统计命中次数和字节数                   | `counter accept`        |
| Limit（限速）                 | 限制匹配速率                       | 防止日志刷屏或缓解简单攻击           |
| Log（日志）                   | 记录匹配的数据包                     | 与 `counter` 组合便于排查问题    |

例如，同时允许多个端口：

```nft
tcp dport {22,80,443} accept
```

这比为每个端口分别写一条规则更加简洁，也更易维护。