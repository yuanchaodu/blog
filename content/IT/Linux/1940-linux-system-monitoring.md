---
title: Linux 系统监控
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoM77
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/194'
---

<img src="images/Linux.svg" width="300">

# Linux 系统监控

Linux 系统监控是指持续采集、分析和展示 Linux 服务器的运行状态，以便及时发现性能瓶颈、资源异常和故障隐患。一个完善的监控体系通常包括**系统资源、进程服务、网络、安全、日志和应用**等多个方面。

下面按照实际运维工作进行介绍。

---

# 一、为什么要做 Linux 系统监控

监控主要解决四类问题：

* **发现故障**

  * CPU占用100%
  * 内存耗尽
  * 磁盘空间满
  * 网络异常

* **定位问题**

  * 哪个进程占CPU最高
  * 哪块磁盘IO繁忙
  * 哪台服务器异常

* **容量规划**

  * CPU是否需要扩容
  * 内存是否足够
  * 磁盘增长趋势

* **预警通知**

  * 微信
  * 邮件
  * 企业微信
  * 钉钉
  * 短信

---

# 二、Linux 需要监控哪些指标

## 1. CPU

重点指标：

* CPU使用率
* Load Average（系统负载）
* CPU等待IO（iowait）
* 上下文切换(Context Switch)
* 中断(Interrupt)

常用命令

```bash
top
htop
mpstat
vmstat
sar -u
```

例如

```bash
top
```

输出

```
%Cpu(s): 10 us, 2 sy, 0 ni, 87 id, 1 wa
```

说明

* us 用户态
* sy 内核态
* id 空闲
* wa IO等待

如果

```
wa > 20%
```

通常意味着磁盘IO存在瓶颈。

---

## 2. 内存

重点关注

* 已用内存
* 空闲内存
* Buffer
* Cache
* Swap
* OOM

命令

```bash
free -h
```

例如

```
              total used free shared buff/cache available
Mem:           32G   20G  2G      1G      10G      11G
```

重点看

```
available
```

而不是

```
free
```

Linux 会利用空闲内存做缓存，因此 free 很少。

---

## 3. 磁盘

包括

* 使用率
* inode
* IO速度
* IO等待
* 延迟

命令

```bash
df -h
```

查看容量

```bash
df -i
```

查看inode

```bash
iostat -x
```

查看IO

重点指标

```
util
await
svctm
```

其中

```
await
```

表示IO平均等待时间。

---

## 4. 网络

监控

* 网卡流量
* 带宽
* TCP连接
* 丢包
* 重传
* Socket数量

命令

```bash
sar -n DEV
```

```bash
ss -s
```

```bash
iftop
```

```bash
ip -s link
```

---

## 5. 文件系统

需要监控

* 磁盘空间
* inode
* 挂载点
* 文件数量

例如

```bash
df -h
```

---

## 6. 系统负载（Load Average）

查看

```bash
uptime
```

例如

```
load average: 2.30 1.90 1.85
```

一般经验：

对于一台

```
8核CPU
```

如果

```
Load > 8
```

说明CPU可能繁忙。

如果

```
CPU使用率低
Load很高
```

多数情况是

* IO等待
* D状态进程

---

## 7. 进程

查看

```bash
ps aux
```

或者

```bash
top
```

重点监控

* CPU最高进程
* 内存最高进程
* Zombie进程
* D状态进程

---

## 8. 服务

例如

```bash
systemctl status nginx
```

监控

* nginx
* mysql
* redis
* docker
* kubelet

---

## 9. 日志

Linux 日志一般位于

```
/var/log/
```

例如

```
messages
secure
cron
dmesg
```

实时查看

```bash
tail -f /var/log/messages
```

---

# 三、常用监控命令

| 功能   | 命令                |
| ---- | ----------------- |
| CPU  | top、mpstat        |
| 内存   | free -h           |
| IO   | iostat            |
| 网络   | sar、iftop         |
| 进程   | ps、top            |
| 登录用户 | w、who             |
| 网络连接 | ss、netstat        |
| 日志   | journalctl        |
| 系统信息 | uname、hostnamectl |
| 实时资源 | htop、glances      |

---

# 四、企业级监控工具

## 1. Prometheus（推荐）

特点

* 开源
* Kubernetes 标准方案
* 时序数据库
* Pull模式
* 支持 Alertmanager

架构

```
Node Exporter
        │
        ▼
 Prometheus
        │
        ▼
 Grafana
```

适用于

* Linux
* Docker
* Kubernetes
* 数据库
* 中间件

---

## 2. Grafana

作用

不是采集工具，而是展示工具。

特点

* 仪表盘
* 报表
* 告警
* 大屏展示

几乎可以连接所有监控系统。

---

## 3. Zabbix

适合

传统企业

特点

* Agent模式
* Web管理
* 自动发现
* 告警完善
* 运维成熟

适用于

* Windows
* Linux
* 网络设备
* 数据库

---

## 4. Nagios

老牌监控系统

优点

* 稳定
* 插件丰富

缺点

* 配置复杂
* 界面较旧

目前新项目较少采用。

---

## 5. Netdata

特点

* 安装简单
* 秒级监控
* 开箱即用
* 自动生成图表

适合

个人服务器、小型项目或临时排查问题。

---

## 6. Glances

安装

```bash
pip install glances
```

运行

```bash
glances
```

可以同时查看

* CPU
* 内存
* IO
* 网络
* 进程

非常适合快速诊断。

---

# 五、监控体系设计

一个典型的企业级 Linux 监控架构如下：

```text
          Linux Server
                │
         Node Exporter
                │
                ▼
          Prometheus
                │
      Alertmanager
                │
      企业微信/邮件/短信
                │
                ▼
            Grafana
```

监控数据由 Exporter 采集，Prometheus 定期抓取并存储；当指标超过阈值时，Alertmanager 负责发送告警；Grafana 则提供统一的可视化展示和历史分析。

---

# 六、告警建议

建议根据业务重要性设置不同等级的告警阈值，避免误报和漏报。

| 指标             | 建议阈值                   |
| -------------- | ---------------------- |
| CPU使用率         | 连续5分钟 > 80%（严重可设 >90%） |
| Load Average   | 持续超过 CPU 核心数           |
| 内存使用率          | > 85%                  |
| Swap使用率        | > 20%（持续增长需关注）         |
| 磁盘使用率          | > 80%，90% 严重           |
| inode使用率       | > 80%                  |
| 磁盘IO等待（iowait） | > 20%                  |
| 文件系统不可写        | 立即告警                   |
| 服务状态           | 服务停止立即告警               |
| 网络丢包率          | > 1% 持续存在需排查           |
| SSH登录失败        | 短时间内大量失败尝试应告警          |

---

# 七、推荐方案（面向企业）

对于企业数字化工厂或生产环境，建议采用以下组合：

* **监控采集**：Prometheus + Node Exporter
* **可视化**：Grafana
* **告警**：Alertmanager（对接企业微信、邮件、短信等）
* **日志管理**：ELK（Elasticsearch + Logstash + Kibana）或 Loki + Promtail
* **系统与应用监控**：数据库、中间件和业务应用分别部署对应 Exporter（如 MySQL Exporter、Redis Exporter、JMX Exporter 等）

这种方案具有开源、扩展性强、生态丰富的特点，已成为云原生和现代企业运维的主流选择。如果是传统数据中心且设备类型较多（Linux、Windows、网络设备等），也可以选择 **Zabbix** 作为统一监控平台；若正在建设 Kubernetes 或微服务平台，则 **Prometheus + Grafana + Alertmanager** 更具优势。对于您所在的化工企业，这类方案还能结合生产系统和网络设备监控，逐步构建统一的 IT 基础设施监控平台，并与 CMDB、ITSM 或告警平台联动，实现故障发现、定位、通知和闭环处理。
