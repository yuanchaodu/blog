---
title: Oracle 整体架构概览
section: IT
category: Oracle
---

# Oracle 整体架构概览

<img src="images/Oracle.svg" width="300">

如果你是想快速建立一个 **Oracle Database 的整体架构认知**，可以把它理解成四层：**客户端 → Instance（实例）→ Database（数据库文件）→ 存储**。

```text
客户端 / 应用程序
      │
      │ SQL / JDBC / OCI / SQL*Net
      ▼
┌─────────────────────────────────────────┐
│           Oracle Instance               │
│                                         │
│  ┌──────────── SGA ─────────────────┐   │
│  │ Shared Pool                      │   │
│  │ Database Buffer Cache            │   │
│  │ Redo Log Buffer                  │   │
│  │ Large / Java / Streams Pool ...  │   │
│  └───────────────────────────────────┘   │
│                                         │
│  Background Processes                   │
│  DBWn / LGWR / CKPT / SMON / PMON ...   │
└───────────────────┬─────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│             Oracle Database             │
│                                         │
│  Data Files                             │
│  Control Files                          │
│  Online Redo Log Files                  │
│  Temp Files                             │
│  Archived Redo Logs                     │
└─────────────────────────────────────────┘
```

### 1. Oracle 最核心的两个概念

**Instance（实例）**不是数据库文件本身，而是：

**Instance = 内存结构 + 后台进程**

例如：

```text
Instance
 ├─ SGA
 │   ├─ Shared Pool
 │   ├─ Buffer Cache
 │   └─ Redo Log Buffer
 │
 └─ Background Processes
     ├─ DBWn
     ├─ LGWR
     ├─ CKPT
     ├─ SMON
     └─ PMON
```

而 **Database（数据库）** 指磁盘上的物理文件：

```text
Database
 ├─ Data Files
 ├─ Control Files
 └─ Redo Log Files
```

所以最经典的一句话就是：

> **Oracle Server = Oracle Instance + Oracle Database**

---

### 2. Oracle 内存架构

Oracle 内存主要分为：

```text
Oracle Memory
├─ SGA
│  ├─ Database Buffer Cache
│  ├─ Shared Pool
│  │  ├─ Library Cache
│  │  └─ Data Dictionary Cache
│  ├─ Redo Log Buffer
│  ├─ Large Pool
│  ├─ Java Pool
│  └─ Streams Pool
│
└─ PGA
   ├─ Sort Area
   ├─ Hash Area
   ├─ Session Memory
   └─ Private SQL Area
```

**SGA 是实例共享内存。** 多个 Session 可以共同使用。最重要的三个区域是：

* **Buffer Cache**：缓存数据块，减少磁盘 I/O。
* **Shared Pool**：缓存 SQL、执行计划、数据字典等。
* **Redo Log Buffer**：暂存数据库修改产生的 redo。

**PGA 是进程私有内存。** 比如排序、Hash Join、Session 状态等通常会使用 PGA。

---

### 3. Oracle 后台进程

几个最核心的后台进程可以这样记：

| 进程       | 主要作用                                 |
| -------- | ------------------------------------ |
| **DBWn** | 把 Buffer Cache 中的脏块写入 Data File      |
| **LGWR** | 把 Redo Log Buffer 写入 Online Redo Log |
| **CKPT** | 负责 Checkpoint，并更新相关文件头信息             |
| **SMON** | Instance Recovery、系统级清理              |
| **PMON** | 清理异常终止的用户进程等                         |
| **ARCn** | ARCHIVELOG 模式下归档 Redo Log            |

其中事务提交最重要的是 **LGWR**。

例如用户执行：

```sql
UPDATE account
SET balance = balance - 100
WHERE id = 1;

COMMIT;
```

大致流程是：

```text
UPDATE
  │
  ├─ 修改 Buffer Cache 中的数据块
  │
  └─ 产生 Redo
          │
          ▼
   Redo Log Buffer
          │
       COMMIT
          │
          ▼
        LGWR
          │
          ▼
   Online Redo Log
          │
          ▼
   返回 Commit 成功
```

一个很重要的点是：

> **COMMIT 并不要求 DBWn 立即把修改后的数据块写入 Data File。**

COMMIT 的关键是对应的 **Redo 被 LGWR 持久化**。

这也是理解 Oracle **性能、事务、恢复机制**的核心。

---

### 4. Oracle 物理存储结构

物理层面主要有三类核心文件：

```text
Oracle Database
│
├── Control File
│
├── Online Redo Log
│   ├── Group 1
│   ├── Group 2
│   └── Group 3
│
└── Data Files
    ├── SYSTEM
    ├── SYSAUX
    ├── USERS
    ├── UNDO
    └── ...
```

**Data File** 存放真正的数据，例如表、索引、Undo 等。

**Control File** 可以理解为数据库的“目录和身份证”，里面记录数据库结构、数据文件、Redo Log、Checkpoint 等关键信息。

**Online Redo Log** 记录数据库修改，是 Oracle Crash Recovery 的关键基础。

---

### 5. Oracle 逻辑存储结构

Oracle 又在物理 Data File 之上设计了一套逻辑结构：

```text
Database
  │
  └─ Tablespace
       │
       └─ Segment
            │
            └─ Extent
                 │
                 └─ Oracle Block
```

关系可以记成：

**Tablespace → Segment → Extent → Block**

比如创建：

```sql
CREATE TABLE employee (
    id   NUMBER,
    name VARCHAR2(100)
);
```

逻辑上：

```text
USERS Tablespace
      │
      ▼
EMPLOYEE Segment
      │
      ├─ Extent 1
      │   ├─ Block
      │   ├─ Block
      │   └─ Block
      │
      └─ Extent 2
```

物理上这些 Block 最终位于某个：

```text
Data File
```

因此：

> **Tablespace 是逻辑概念，Data File 是物理概念。**

---

### 6. SQL 执行时，各组件怎么协作

例如：

```sql
SELECT *
FROM employees
WHERE employee_id = 100;
```

大致过程：

```text
Client
  │
  ▼
Server Process
  │
  ├─ SQL 是否已经解析？
  │       │
  │       ▼
  │   Shared Pool
  │
  ├─ 是否已有数据块？
  │       │
  │       ▼
  │   Buffer Cache
  │       │
  │       └─ 没有
  │           │
  │           ▼
  │        Data File
  │
  ▼
返回结果
```

因此 Oracle 优化里经常出现：

**Hard Parse / Soft Parse、Library Cache、Buffer Cache Hit Ratio、Physical Read、Logical Read**

这些概念其实都可以放回这张架构图里理解。

---

### 7. Undo 和 Redo 很容易混淆

可以简单记：

**Redo = 怎么重新做。**

**Undo = 怎么撤销。**

例如：

```sql
UPDATE employee
SET salary = 10000
WHERE id = 10;
```

Oracle 会产生：

```text
Redo
→ 记录修改所需的信息
→ 用于 Crash Recovery

Undo
→ 保存旧版本信息
→ 用于 Rollback
→ 用于 Read Consistency
```

因此：

```text
Redo
主要解决：数据库恢复

Undo
主要解决：事务回滚 + 一致性读
```

---

### 8. Oracle 现代架构还要理解 CDB / PDB

现代 Oracle Database 中还非常重要的一层是 **Multitenant Architecture**：

```text
Oracle Instance
       │
       ▼
      CDB
       │
       ├── CDB$ROOT
       │
       ├── PDB$SEED
       │
       ├── PDB_APP1
       │
       ├── PDB_APP2
       │
       └── PDB_APP3
```

其中：

**CDB（Container Database）** 是容器数据库。

**PDB（Pluggable Database）** 是可插拔数据库，业务数据库通常部署在 PDB 中。

所以现在学习 Oracle，建议把整体体系记成：

```text
Client
   ↓
Listener
   ↓
Oracle Instance
   ├─ SGA
   ├─ PGA
   └─ Background Processes
   ↓
CDB
   ├─ Root
   └─ PDB
       ↓
Tablespace
       ↓
Segment
       ↓
Extent
       ↓
Block
       ↓
Data File

同时：
Redo → Online Redo Log → Archive Log
Undo → Undo Tablespace
Control File → 保存数据库结构与状态信息
```

如果是为了 **DBA / OCP / Oracle 面试** 来理解架构，最值得先吃透的一条主线是：

**SQL 执行 → Shared Pool → Buffer Cache → Undo/Redo → LGWR/DBWn → Redo Log/Data File → Checkpoint/Recovery。**

把这条链理解透之后，Oracle 的 **事务、性能优化、备份恢复、RAC、Data Guard** 都会容易很多。