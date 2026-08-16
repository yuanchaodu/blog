---
title: Oracle 核心特性
section: IT
category: Oracle
<<<<<<< HEAD
discussion_id: D_kwDOS1Ul_s4Aoicr
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/205'
=======
>>>>>>> 316994b (docs: add Oracle features)
---

# Oracle 核心特性

<img src="images/Oracle.svg" width="300">

<<<<<<< HEAD
如果你指的是 **Oracle Database 的核心特性**，可以从下面几个方面理解：

=======
>>>>>>> 316994b (docs: add Oracle features)
1. **企业级关系型数据库**

   * 基于关系模型，使用 SQL。
   * 支持复杂查询、视图、索引、存储过程、触发器等。
   * 强调大型企业系统中的稳定性、可靠性和数据一致性。

2. **事务与 ACID**

   * 完整支持事务的 **原子性、一致性、隔离性、持久性（ACID）**。
   * 提供提交 `COMMIT`、回滚 `ROLLBACK`、保存点 `SAVEPOINT`。
   * 通过多版本并发控制等机制，在高并发情况下兼顾一致性和性能。

3. **PL/SQL**

   * Oracle 自带的过程化 SQL 扩展语言。
   * 支持变量、条件、循环、异常处理、函数、存储过程和 Package。
   * 可以把业务逻辑放到数据库端执行。

4. **高可用与容灾**

   * **RAC（Real Application Clusters）**：多个数据库实例共同访问数据库，提高可用性和扩展能力。
   * **Data Guard**：建立主库/备库体系，用于灾难恢复和高可用。
   * 支持备份恢复、故障切换等企业级机制。

5. **强大的并发控制**

   * Oracle 的一个重要特点是 **读操作通常不会阻塞写操作，写操作通常也不会阻塞读操作**。
   * 利用 Undo 和一致性读（Consistent Read）机制，让查询看到符合特定时间点的一致数据版本。
   * 支持不同事务隔离级别。

6. **完善的存储体系**

   * 逻辑结构：**Tablespace → Segment → Extent → Block**。
   * 物理结构包括 Data File、Control File、Redo Log 等。
   * 支持分区表、索引组织表、LOB 等多种存储方式。

7. **Redo 与 Undo**

   * **Redo**：记录数据库发生的修改，主要用于实例恢复和数据保护。
   * **Undo**：保存修改前的数据版本，用于事务回滚、一致性读等。
   * 这是理解 Oracle 事务、恢复和 MVCC 的关键。

8. **安全性**

   * 提供用户、角色、系统权限和对象权限。
   * 支持审计、加密、细粒度访问控制等企业安全能力。

9. **性能优化能力**

   * 基于成本的优化器 **CBO（Cost-Based Optimizer）**。
   * 根据统计信息选择执行计划。
<<<<<<< HEAD
   * 提供 Explain Plan、AWR、ASH 等诊断与性能分析机制。

如果是为了**面试/考试**，最值得重点掌握的一条主线是：

**Oracle 架构 → Instance/Database → SGA/PGA → Process → Tablespace/Datafile → Redo/Undo → Transaction → Lock/MVCC → Index → SQL 执行计划 → RAC/Data Guard。**

这条线基本覆盖 Oracle 的核心知识体系。
=======
   * 提供 Explain Plan、AWR、ASH 等诊断与性能分析机制。
>>>>>>> 316994b (docs: add Oracle features)
