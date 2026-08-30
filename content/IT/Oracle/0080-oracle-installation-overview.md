---
title: Oracle 安装流程
section: IT
category: Oracle
discussion_id: D_kwDOS1Ul_s4Ao3-n
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/206'
---

# Oracle 安装流程

<img src="images/Oracle.svg" width="300">

**Oracle Database 数据库的安装流程**，一般可以按下面这套思路来做。Oracle 官方目前不同版本、不同系统的安装方式会有差异；例如 Windows 常见企业环境仍大量使用 19c，而 Linux 上也可以安装更新的 Oracle AI Database / 23ai。([Oracle Docs][1])

### Oracle 安装总体流程

1. **确认环境**

   * 操作系统：Windows / Linux
   * Oracle 版本：例如 19c、23ai
   * 内存、磁盘空间是否满足要求
   * 主机名、IP 是否配置正常
   * 确定数据库名称，例如：

     * SID：`ORCL`
     * Service Name：`orcl`
     * PDB：`ORCLPDB`

2. **下载 Oracle 安装包**
   从 Oracle 官方下载对应操作系统和版本的 Database 安装包。

3. **准备安装目录**

   Windows 常见目录例如：

   ```text
   C:\app\oracle
   C:\app\oracle\product\19.0.0\dbhome_1
   ```

   Linux 常见目录例如：

   ```bash
   /u01/app/oracle
   /u01/app/oraInventory
   /u01/app/oracle/product/19.0.0/dbhome_1
   ```

   Oracle 官方 Linux 安装流程通常要求使用 `oracle` 用户作为软件所有者，并预先准备 Oracle Base、Inventory 等目录。([Oracle Docs][2])

4. **启动安装程序**

   Windows：

   ```text
   setup.exe
   ```

   建议右键：

   ```text
   以管理员身份运行
   ```

   Linux 通常进入 Oracle Home 后执行：

   ```bash
   ./runInstaller
   ```

5. **选择安装模式**

   常见选项是：

   ```text
   Create and configure a single instance database
   ```

   即：

   **安装 Oracle 软件 + 同时创建数据库**

   如果是服务器正式环境，也经常选择：

   ```text
   Set Up Software Only
   ```

   先安装软件，再通过 DBCA 单独创建数据库。

6. **配置数据库**

   典型配置例如：

   ```text
   Global Database Name: orcl
   SID: ORCL
   ```

   新版本 Oracle 通常采用 CDB/PDB 架构，例如：

   ```text
   CDB: ORCL
   PDB: ORCLPDB
   ```

   同时设置管理员账号密码：

   ```text
   SYS
   SYSTEM
   PDBADMIN
   ```

7. **配置存储位置**

   例如：

   ```text
   Oracle Base:
   C:\app\oracle

   Software Location:
   C:\app\oracle\product\19.0.0\dbhome_1

   Database Files:
   C:\app\oracle\oradata
   ```

   Linux 则常见：

   ```text
   /u01/app/oracle/oradata
   ```

8. **执行安装**

   安装程序会完成：

   ```text
   Oracle Database Software
           ↓
   Listener
           ↓
   Database
           ↓
   Data Dictionary
           ↓
   CDB / PDB
   ```

   Linux 安装过程中通常还会提示使用 `root` 执行相关配置脚本。([Oracle Docs][2])

9. **检查监听器**

   安装完成后检查：

   ```bash
   lsnrctl status
   ```

   正常情况下会看到类似：

   ```text
   PORT = 1521
   SERVICE_NAME = orcl
   ```

   Oracle 默认监听端口通常为：

   ```text
   1521
   ```

10. **测试数据库连接**

使用 SQL*Plus：

```bash
sqlplus / as sysdba
```

查看数据库状态：

```sql
select status from v$instance;
```

正常应该返回：

```text
OPEN
```

查看数据库：

```sql
select name, open_mode from v$database;
```

查看 PDB：

```sql
show pdbs;
```

### 可以简单记成

```text
下载安装包
    ↓
检查系统环境
    ↓
创建 Oracle 安装目录
    ↓
运行 setup.exe / runInstaller
    ↓
安装 Oracle Database 软件
    ↓
配置 SID / CDB / PDB
    ↓
创建数据库
    ↓
配置 Listener 1521
    ↓
设置环境变量
    ↓
SQL*Plus 登录测试
    ↓
安装完成
```

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/ "Oracle Database Database Installation Guide, 19c for Microsoft Windows"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/21/ladbi/running-oracle-universal-installer-to-install-oracle-database.html "Running Oracle Database Setup Wizard to Install Oracle Database"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/install-oracle-database.html "Installing the Oracle Database Software"
