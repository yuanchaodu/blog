---
title: Oracle 安装目录结构
section: IT
category: Oracle
---

# Oracle 安装目录结构

<img src="images/Oracle.svg" width="300">

Oracle Database 安装后的目录结构，核心可以先记住两个概念：**Oracle Base** 和 **Oracle Home**。不同版本、Linux/Windows 路径会有差异，但整体结构基本一致。

以 Linux 上常见安装方式为例：

```text
/u01/app/oracle/                    ← ORACLE_BASE
├── admin/
│   └── orcl/                      ← 数据库管理目录
│       ├── adump/                 ← 审计日志
│       ├── dpdump/                ← Data Pump 导入导出目录
│       └── pfile/                 ← 参数文件相关
│
├── diag/                          ← 诊断信息目录
│   └── rdbms/
│       └── orcl/
│           └── orcl/
│               ├── alert/         ← XML 格式告警信息
│               ├── trace/         ← alert.log、trace 文件
│               ├── incident/      ← 故障事件
│               └── cdump/         ← core dump
│
├── product/
│   └── 19.0.0/
│       └── dbhome_1/              ← ORACLE_HOME
│           ├── bin/               ← Oracle 可执行程序
│           ├── dbs/               ← 参数文件、密码文件
│           ├── network/
│           │   └── admin/         ← listener.ora、tnsnames.ora
│           ├── rdbms/             ← 数据库核心组件
│           ├── sqlplus/           ← SQL*Plus
│           ├── jdbc/              ← JDBC 驱动
│           ├── lib/               ← 库文件
│           ├── OPatch/            ← 补丁工具
│           └── inventory/         ← 部分安装信息
│
└── oraInventory/                  ← Oracle 软件清单
```

### 1. ORACLE_BASE

例如：

```bash
ORACLE_BASE=/u01/app/oracle
```

它是 Oracle 软件、日志、诊断信息等内容的基础目录。

可以查看：

```bash
echo $ORACLE_BASE
```

### 2. ORACLE_HOME

例如 Oracle 19c：

```bash
ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
```

这是 **Oracle 数据库软件真正安装的位置**。

最常用的目录是：

| 目录                           | 作用                                   |
| ---------------------------- | ------------------------------------ |
| `$ORACLE_HOME/bin`           | sqlplus、rman、lsnrctl、expdp 等命令       |
| `$ORACLE_HOME/dbs`           | spfile、pfile、密码文件等                   |
| `$ORACLE_HOME/network/admin` | listener.ora、tnsnames.ora、sqlnet.ora |
| `$ORACLE_HOME/rdbms`         | Oracle RDBMS 核心组件                    |
| `$ORACLE_HOME/lib`           | Oracle 库文件                           |
| `$ORACLE_HOME/jdbc`          | JDBC 驱动                              |
| `$ORACLE_HOME/OPatch`        | Oracle 补丁管理工具                        |

例如：

```bash
$ORACLE_HOME/bin/sqlplus
$ORACLE_HOME/bin/rman
$ORACLE_HOME/bin/lsnrctl
```

### 3. 数据文件目录

数据库的数据文件一般**不一定放在 ORACLE_HOME 中**。

常见结构：

```text
/u01/app/oracle/oradata/
└── ORCL/
    ├── system01.dbf
    ├── sysaux01.dbf
    ├── undotbs01.dbf
    ├── users01.dbf
    ├── temp01.dbf
    └── redo*.log
```

对应：

```text
oradata
   │
   └── ORCL
       ├── system01.dbf     SYSTEM 表空间
       ├── sysaux01.dbf     SYSAUX 表空间
       ├── undotbs01.dbf    UNDO 表空间
       └── users01.dbf      USERS 表空间
```

实际数据文件位置可以通过 SQL 查询：

```sql
SELECT name FROM v$datafile;
```

临时文件：

```sql
SELECT name FROM v$tempfile;
```

控制文件：

```sql
SELECT name FROM v$controlfile;
```

### 4. `$ORACLE_HOME/dbs`

Linux 下这个目录很重要：

```text
$ORACLE_HOME/dbs/
├── spfileORCL.ora
├── initORCL.ora
├── orapwORCL
└── hc_ORCL.dat
```

其中：

```text
spfileORCL.ora
```

是服务器参数文件 SPFILE。

```text
initORCL.ora
```

是文本参数文件 PFILE。

```text
orapwORCL
```

是数据库密码文件，例如用于：

```sql
sqlplus / as sysdba
```

以及远程 SYSDBA 认证。

### 5. 网络配置目录

位置通常是：

```bash
$ORACLE_HOME/network/admin
```

里面最常见三个文件：

```text
network/admin/
├── listener.ora
├── tnsnames.ora
└── sqlnet.ora
```

分别负责：

```text
listener.ora
    ↓
监听器配置

tnsnames.ora
    ↓
客户端连接数据库的服务名配置

sqlnet.ora
    ↓
Oracle Net 行为、认证方式等配置
```

### 6. 日志与诊断目录

Oracle 11g 以后主要使用 ADR（Automatic Diagnostic Repository）。

例如：

```text
$ORACLE_BASE/diag/rdbms/orcl/orcl/
```

重点目录：

```text
trace/
```

通常可以找到：

```text
alert_ORCL.log
*.trc
*.trm
```

也就是经常排查数据库故障时看的 **alert 日志**。

可以在数据库中查实际位置：

```sql
SELECT value
FROM v$diag_info
WHERE name = 'Diag Trace';
```

### 7. Windows 上的典型结构

Windows 可能类似：

```text
C:\app\oracle\
├── product\
│   └── 19.0.0\
│       └── dbhome_1\
│           ├── bin\
│           ├── database\
│           ├── network\
│           │   └── admin\
│           ├── rdbms\
│           └── ...
│
├── admin\
├── diag\
└── oradata\
    └── ORCL\
```

一个明显区别是 Linux 常用：

```text
$ORACLE_HOME/dbs
```

Windows 对应很多配置文件会放在：

```text
%ORACLE_HOME%\database
```

---

如果从 DBA 运维角度记忆，最重要的其实就是这几个：

```text
ORACLE_BASE
│
├── product/.../dbhome_1        ← Oracle 软件
│   ├── bin                     ← 命令
│   ├── dbs                     ← SPFILE / 密码文件
│   └── network/admin           ← 监听和 TNS
│
├── oradata                     ← 数据文件
│
├── admin                       ← 管理文件
│
└── diag                        ← alert.log / trace
```