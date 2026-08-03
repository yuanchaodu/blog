---
title: Oracle 发展历史与版本演进
section: IT
category: Oracle
---

# Oracle 数据库发展历史与版本演进

<img src="images/Oracle.svg" width="300">

Oracle Database 是全球应用最广泛的企业级关系数据库之一，也是关系数据库（RDBMS）发展的重要代表。从 1977 年创立至今，Oracle 的发展经历了**关系数据库时代、互联网时代、网格计算时代、云计算时代以及 AI 数据库时代**五个主要阶段。([Oracle Docs][1])

---

## 一、Oracle 发展历史

### 第一阶段：关系数据库诞生（1977～1988）

1977 年，Larry Ellison、Bob Miner 和 Ed Oates 创立 Software Development Laboratories（SDL），后来更名为 Relational Software Inc.（RSI），最终发展为 Oracle Corporation。Oracle 的目标是将 IBM 提出的关系数据库理论商业化。([Oracle Software Downloads][2])

### Oracle V2（1979）

> 为什么没有 V1？
>
> Larry Ellison 认为客户不会愿意购买“Version 1”的软件，因此第一款正式发布的产品直接命名为 **Oracle Version 2**。

特点：

* 全球第一款商业 SQL 关系数据库
* 支持 SQL 查询
* 面向企业数据管理

意义：

Oracle 成为世界第一家商业化关系数据库公司，为后来数据库行业的发展奠定了基础。([Oracle Docs][1])

---

### Oracle V3（1983）

主要改进：

* 使用 C 语言重写数据库核心
* 实现跨平台运行
* 支持 UNIX、IBM Mainframe 等多个平台

这是 Oracle 真正快速发展的开始。

---

### Oracle V4（1984）

首次引入：

* Multi-Version Read Consistency（MVCC，多版本一致性读）

今天 Oracle 一直使用的**读写不阻塞机制**正是源于这一版本。

---

### Oracle V5（1985）

进入 Client/Server（客户端/服务器）时代。

新增：

* 分布式数据库
* SQL*Net 网络通信
* 多节点访问

Oracle 开始支持企业网络化应用。

---

### Oracle V6（1988）

Oracle 第一次真正成为企业级数据库。

主要新增：

* 行级锁（Row Lock）
* 在线备份恢复
* PL/SQL 初版
* 性能大幅提升

很多今天仍在使用的 Oracle 核心机制都始于 V6。([Oracle Software Downloads][2])

---

# 第二阶段：企业数据库成熟（1992～1999）

## Oracle7（1992）

这是 Oracle 历史上最经典的版本之一。

新增：

* 存储过程（Stored Procedure）
* Trigger（触发器）
* Cost Based Optimizer（CBO）
* Referential Integrity（外键）

Oracle 从"数据库"变成了"数据库开发平台"。

---

## Oracle8（1997）

Oracle 提出了 Object-Relational Database（对象关系数据库）。

新增：

* 对象类型
* LOB（大对象）
* Partition（分区表）

企业开始管理 TB 级数据。

这是 Oracle 第一次重点强调：

> 大数据（Big Database）

而不是今天所说的大数据（Big Data）。

---

## Oracle8i（1999）

这里第一次出现：

**i = Internet**

Oracle 全面拥抱互联网。

新增：

* Java VM
* XML
* Internet Protocol
* JDBC
* Web Application

Oracle 官方称：

> Internet Database

这一版本是互联网时代的重要里程碑。([Oracle Software Downloads][2])

---

# 第三阶段：互联网与网格计算（2001～2012）

## Oracle9i（2001）

继续强化 Internet。

新增：

* Real Application Cluster（RAC）
* Data Guard
* Flashback 雏形

RAC 是 Oracle 最具代表性的高可用技术之一。

---

## Oracle10g（2003）

这里：

**g = Grid Computing**

Oracle 提出了 Grid Computing（网格计算）。

核心理念：

很多普通服务器组成一个大型数据库。

新增：

* ASM（Automatic Storage Management）
* Automatic Database Diagnostic Monitor（ADDM）
* AWR
* Automatic Memory Management

Oracle 开始强调：

> 自动管理数据库（Self-managing Database）

---

## Oracle11g（2007）

Oracle DBA 最经典的一代。

新增：

* Active Data Guard
* SecureFiles
* Result Cache
* SQL Plan Management
* Database Replay
* Real Application Testing

11g 的稳定性极高，因此至今仍有不少企业使用 11.2.0.4。([Wikipedia][3])

---

# 第四阶段：云计算时代（2013～2021）

## Oracle12c（2013）

这里：

**c = Cloud**

Oracle 正式进入云计算时代。

最重要创新：

### 1、Multitenant Architecture（多租户）

一个数据库：

```
CDB
 ├── PDB1
 ├── PDB2
 ├── PDB3
```

一套数据库实例可运行多个独立数据库。

这成为后来 Oracle Cloud 的基础。

---

### 2、In-Memory Database

新增：

* 内存列存储
* Hybrid Columnar Compression

OLTP 与 OLAP 可在同一数据库中运行。

---

### Oracle18c（2018）

Oracle 放弃传统版本号（如 12.3），改为按年份命名。

实际上：

```
12.3
↓

18c
```

因此：

18c 本质上属于 12c 系列，但采用新的年度版本命名方式。([Oracle Docs][4])

主要特点：

* Autonomous Database 基础
* 自动补丁
* 自动优化

---

### Oracle19c（2019）

Oracle 官方定位：

> Long Term Release（长期支持版本）

这是目前企业应用最广泛的 Oracle 版本之一。

特点：

* 稳定
* 成熟
* 长期支持
* 适合作为生产环境

大量企业目前仍运行 Oracle 19c。([Wikipedia][3])

---

### Oracle21c（2021）

Oracle 将其定义为：

Innovation Release（创新版本）

新增：

* Blockchain Table
* Immutable Table
* SQL Macro
* Native JSON
* AutoML

主要供用户提前体验新特性。

---

# 第五阶段：AI 数据库时代（2023～至今）

## Oracle23ai（原 Oracle 23c）

Oracle 将 Oracle23c 更名为 **Oracle Database 23ai**，强调数据库对 AI 工作负载和向量数据处理的支持。

主要新增：

* AI Vector Search（向量检索）
* JSON Relational Duality（JSON 与关系模型双向映射）
* SQL Domains
* Property Graph 增强
* 更完善的开发者体验

这一版本标志着 Oracle 从传统关系数据库迈向 AI 原生数据库。([Wikipedia][3])

---

## Oracle AI Database 26ai（2025 起）

Oracle 从 23ai 的后续更新开始推出 **Oracle AI Database 26ai** 命名体系，进一步强化 AI 能力，并继续采用年度版本编号和季度 Release Update（RU）机制。([Oracle Docs][5])

---

# Oracle 版本命名演进

| 时期        | 版本              | 命名含义     | 代表特征                            |
| --------- | --------------- | -------- | ------------------------------- |
| 1979～1997 | V2～8            | 数字版本     | 关系数据库、PL/SQL、对象关系模型             |
| 1999～2002 | 8i、9i           | Internet | 互联网应用、Java、XML                  |
| 2003～2012 | 10g、11g         | Grid     | 网格计算、RAC、ASM、自动管理               |
| 2013～2021 | 12c、18c、19c、21c | Cloud    | 多租户、云架构、自治数据库                   |
| 2023～至今   | 23ai、26ai       | AI       | 向量搜索、AI、JSON Relational Duality |

---

# Oracle 技术演进主线

| 阶段              | 核心能力                   |
| --------------- | ---------------------- |
| V2～V6           | 商业关系数据库、SQL、事务、并发控制    |
| Oracle7         | 企业开发平台（PL/SQL、触发器、优化器） |
| Oracle8/8i      | 对象关系数据库、互联网支持          |
| Oracle9i/10g    | 高可用（RAC）、网格计算、自动化运维    |
| Oracle11g       | 企业级稳定、高性能、高可用          |
| Oracle12c       | 云计算、多租户架构              |
| Oracle19c       | 长期支持（LTS）、成熟稳定         |
| Oracle23ai/26ai | AI 数据库、向量检索、JSON 与关系融合 |

---

# Oracle 版本号规则

Oracle 的版本号规则也经历了变化：

* **12c 及以前**：采用 `主版本.发布版.修订版.补丁.平台` 的形式，例如 `12.2.0.1.0`。
* **18c 起**：采用年度版本号，如 **18c、19c、21c、23ai、26ai**，并通过季度 **Release Update（RU）** 持续更新。例如 `19.27.0.0.0` 表示 Oracle 19 的第 27 个季度更新。Oracle 目前已取消过去的 Patch Set 模式，统一采用 RU（以及后续的 MRP）进行维护。([Oracle Docs][6])

## 总结

Oracle 数据库近五十年的演进，体现了企业计算技术的发展方向：

* **1979～1990 年代**：建立商业关系数据库基础，完善事务、并发控制和 PL/SQL。
* **2000 年代**：聚焦互联网、高可用和网格计算，以 RAC、ASM 等技术提升企业级能力。
* **2010 年代**：面向云计算，推出多租户架构和自治数据库，为云原生部署奠定基础。
* **2020 年代以来**：进入 AI 数据库时代，围绕向量搜索、AI 应用和开发者体验持续演进，数据库已从单纯的数据存储平台发展为智能数据平台。

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/introduction-to-oracle-database.html?source=%3Aex%3Apw%3A%3A%3A%3A%3ATNS_SQL_2_D "Introduction to Oracle Database"
[2]: https://download.oracle.com/docs/en/database/oracle/oracle-database/21/cncpt/database-concepts.pdf "Oracle® Database"
[3]: https://en.wikipedia.org/wiki/Oracle_Database "Oracle Database"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/18/upgrd/oracle-database-release-numbers.html?source=%3Aso%3Atw%3Aor%3Aawr%3Aore%3A%3A%3Aautonmousblog "About Oracle Database Release Numbers"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/26/upgrd/oracle-database-release-numbers.html?source=docs "About Oracle Database Release Numbers"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/21/upgrd/oracle-database-release-numbers.html "About Oracle Database Release Numbers"