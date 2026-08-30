---
title: Oracle 授权模式简介
section: IT
category: Oracle
---

# Oracle 授权模式简介

<img src="images/Oracle.svg" width="300">

Oracle 数据库授权（Licensing）主要按**处理能力、用户数量、部署方式和数据库版本**来计费。企业实际采购时，最常见的是以下几种模式：


| 授权模式                      | 计量方式                     | 适用场景                          |
| ------------------------- | ------------------------ | ----------------------------- |
| **Processor License**     | 按服务器处理器/核心数计算            | 用户数量很多或无法准确统计，如互联网业务、大型生产系统   |
| **Named User Plus (NUP)** | 按指定用户/设备数量计算，同时受最低采购数量限制 | 用户明确、规模较小的内部系统                |
| **Cloud Subscription**    | 按云服务实际订阅的计算资源/服务规格计费     | Oracle Cloud / SaaS / PaaS 场景 |
| **特殊协议**                  | 如 ULA 等企业级协议             | 大型集团、Oracle 产品使用规模较大的企业       |

### 1. Processor 授权

这是大型 Oracle Database 环境中常见的授权方式。

对于多核 CPU，通常不是简单地按照“物理 CPU 数”计算，而需要结合 **Processor Core Factor**：

**所需 Processor License ≈ 物理核心数 × Core Factor**

例如，假设某服务器有：

> 2 CPU × 16 Core = 32 个物理核心
> Core Factor = 0.5
> → 需要约 **16 Processor Licenses**

具体计算仍取决于处理器型号、部署架构、合同条款及 Oracle 当期适用的 Core Factor 规则。

### 2. Named User Plus（NUP）

NUP 按可以访问 Oracle 数据库的**用户或设备**授权。

需要特别注意，Oracle 授权中的“用户”通常不能简单理解成数据库账号。例如一个 ERP 系统只有一个中间件账号连接 Oracle，但后面有 500 个员工使用 ERP，授权评估通常需要考虑这些最终用户，而不能只计算那个数据库连接账号。

此外，NUP 通常还有与 Processor 数量相关的**最低采购数量**要求，因此不能简单地说“系统只有 10 个用户，就买 10 个 NUP”。

### 3. Edition 和 Options 也很重要

Oracle Database 不同 Edition 的授权范围不同，例如 Enterprise Edition 与其他版本的功能和许可条件并不相同。

尤其需要注意 Enterprise Edition 上的某些 **Options / Management Packs**。例如某些高级功能可能需要额外授权，而不是购买 Database Enterprise Edition 后自动全部获得许可。

因此实际 License Review 往往需要检查：

**Database Edition → Processor/NUP 数量 → Options/Packs → RAC/虚拟化架构 → DR/Standby 环境 → Cloud 使用情况。**

### 4. 虚拟化是 Oracle 授权的高风险区域

VMware、虚拟机集群、容器、云环境中的 Oracle 授权边界可能比“这个 VM 分配了几个 vCPU”复杂得多。

尤其在大型 VMware 环境中，**技术上限制了多少 CPU**和**Oracle 合同/授权规则认可多少 CPU**不一定是一回事。因此做授权评估时，应把“技术架构”和“合同许可范围”分开分析。

### 5. 一个简单的判断方法

如果是企业内部少量、明确用户，可以先评估 **NUP**；如果用户数量很多、外部用户访问、无法可靠统计最终用户，则通常更倾向 **Processor**。

而大型企业通常还需要进一步考虑 **ULA（Unlimited License Agreement）**、云订阅以及企业级协议。

需要强调的是，Oracle 授权最终以客户的 **Ordering Document、Oracle Master Agreement/OLSA 以及适用的 licensing definitions/政策文件**为准；网上常见的 Core Factor、VMware 或 DR 授权规则不能替代具体合同条款。