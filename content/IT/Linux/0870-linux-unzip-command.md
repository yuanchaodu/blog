---
title: Linux unzip命令
section: IT
category: Linux
---

# Linux unzip命令

<img src="images/Linux.svg" width="300">

`unzip` 是 Linux 中用于解压 `.zip` 压缩文件的常用命令。

## 1. 安装 unzip

部分 Linux 发行版默认没有安装，需要先安装：

### CentOS/RHEL

```bash
yum install unzip
```

或（CentOS 8+/Rocky Linux/AlmaLinux）

```bash
dnf install unzip
```

### Ubuntu/Debian

```bash
apt install unzip
```

---

## 2. 基本用法

### 解压当前目录

```bash
unzip file.zip
```

例如：

```bash
unzip backup.zip
```

解压后文件会放到当前目录。

---

### 解压到指定目录

```bash
unzip file.zip -d 目标目录
```

例如：

```bash
unzip backup.zip -d /data/backup
```

解压到：

```text
/data/backup
```

---

## 3. 查看压缩包内容

不解压，仅查看文件列表：

```bash
unzip -l file.zip
```

例如：

```bash
unzip -l backup.zip
```

输出示例：

```text
Archive: backup.zip
Length    Date    Time    Name
------    ----    ----    ----
1024      ...     ...     test.txt
2048      ...     ...     data.sql
```

---

## 4. 覆盖已有文件

### 自动覆盖

```bash
unzip -o file.zip
```

例如：

```bash
unzip -o backup.zip
```

参数说明：

```text
-o = overwrite
```

直接覆盖同名文件，不询问。

---

### 解压时不覆盖

```bash
unzip -n file.zip
```

例如：

```bash
unzip -n backup.zip
```

已有文件保留不变。

---

## 5. 查看压缩包信息

```bash
unzip -v file.zip
```

例如：

```bash
unzip -v backup.zip
```

可查看：

* 文件大小
* 压缩率
* CRC校验值
* 文件时间

---

## 6. 解压密码保护的 ZIP

```bash
unzip -P 密码 file.zip
```

例如：

```bash
unzip -P 123456 backup.zip
```

> 注意：密码会出现在命令历史记录中，不建议在生产环境使用。

更安全的方式：

```bash
unzip backup.zip
```

然后按提示输入密码。

---

## 7. 只解压指定文件

例如压缩包内容：

```text
backup.zip
├── a.txt
├── b.txt
└── c.txt
```

只解压 `a.txt`：

```bash
unzip backup.zip a.txt
```

---

## 8. 排除指定文件

例如不解压 `.log` 文件：

```bash
unzip backup.zip -x "*.log"
```

---

## 9. 常用组合示例

### 查看压缩包内容

```bash
unzip -l package.zip
```

### 解压到指定目录并覆盖

```bash
unzip -o package.zip -d /opt/app
```

### 静默解压（适合脚本）

```bash
unzip -oq package.zip -d /opt/app
```

参数：

```text
-o  覆盖文件
-q  安静模式
```

---

## 10. 企业运维中常见场景

### 发布应用

```bash
unzip -oq app.zip -d /opt/tomcat/webapps/
```

### 解压备份文件

```bash
mkdir -p /backup/20260616
unzip db_backup.zip -d /backup/20260616
```

### 查看 ZIP 内容确认版本

```bash
unzip -l app.zip | grep version
```

---

## 常见错误

### 错误1：命令不存在

```bash
unzip: command not found
```

解决：

```bash
yum install unzip
```

或

```bash
apt install unzip
```

---

### 错误2：中文文件名乱码

可以尝试：

```bash
unzip -O GBK file.zip
```

或者：

```bash
unzip -O CP936 file.zip
```

适用于 Windows 环境下生成的 ZIP 文件。

---

### 常用速查表

| 功能      | 命令                          |
| ------- | --------------------------- |
| 解压文件    | `unzip file.zip`            |
| 解压到指定目录 | `unzip file.zip -d dir`     |
| 查看内容    | `unzip -l file.zip`         |
| 自动覆盖    | `unzip -o file.zip`         |
| 不覆盖     | `unzip -n file.zip`         |
| 静默解压    | `unzip -q file.zip`         |
| 解压指定文件  | `unzip file.zip filename`   |
| 排除文件    | `unzip file.zip -x "*.log"` |
| 查看详细信息  | `unzip -v file.zip`         |
| 解压加密ZIP | `unzip file.zip`（输入密码）      |