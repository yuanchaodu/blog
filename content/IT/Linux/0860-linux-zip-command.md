---
title: Linux zip 命令
section: IT
category: Linux
---

# Linux zip 命令

<img src="images/Linux.svg" width="300">

`zip` 是 Linux 中最常用的压缩命令之一，用于将文件或目录打包成 `.zip` 格式。

## 1. 安装 zip

不同发行版安装方法：

### CentOS / Rocky / AlmaLinux

```bash
sudo yum install zip unzip
```

### RHEL 8/9

```bash
sudo dnf install zip unzip
```

### Ubuntu / Debian

```bash
sudo apt install zip unzip
```

---

## 2. 压缩单个文件

```bash
zip test.zip test.txt
```

生成：

```text
test.zip
```

包含：

```text
test.txt
```

---

## 3. 压缩多个文件

```bash
zip files.zip file1.txt file2.txt file3.txt
```

---

## 4. 压缩目录

使用 `-r`（递归）参数：

```bash
zip -r project.zip project/
```

例如：

```bash
zip -r backup.zip /data/app
```

将整个 `app` 目录及其子目录、文件全部压缩。

---

## 5. 排除指定文件

使用 `-x`：

```bash
zip -r project.zip project -x "*.log"
```

排除所有 `.log` 文件。

排除多个类型：

```bash
zip -r project.zip project \
    -x "*.log" "*.tmp" "*.bak"
```

---

## 6. 指定压缩等级

压缩等级：

* `-0` 不压缩，仅打包
* `-1` 最快压缩
* `-9` 最高压缩率

例如：

```bash
zip -9 -r backup.zip /data
```

---

## 7. 查看压缩包内容

```bash
unzip -l backup.zip
```

输出示例：

```text
Archive: backup.zip

Length      Date    Time    Name
--------  ---------- -----   ----
1024      2025-01-01 10:00   file1.txt
2048      2025-01-01 10:01   file2.txt
```

---

## 8. 解压 ZIP 文件

### 解压到当前目录

```bash
unzip backup.zip
```

### 解压到指定目录

```bash
unzip backup.zip -d /tmp/test
```

---

## 9. 加密 ZIP 文件

创建密码保护压缩包：

```bash
zip -e secret.zip file.txt
```

执行后提示输入密码：

```text
Enter password:
Verify password:
```

也可以：

```bash
zip -er secret.zip mydir
```

加密整个目录。

> 注意：传统 ZIP 加密安全性一般，不建议用于高安全场景。

---

## 10. 更新压缩包

将修改后的文件更新到已有压缩包：

```bash
zip -u backup.zip file.txt
```

---

## 11. 删除压缩包中的文件

```bash
zip -d backup.zip file.txt
```

---

## 常用场景示例

### 备份应用目录

```bash
zip -9 -r app_backup_$(date +%F).zip /opt/app
```

### 压缩日志并排除临时文件

```bash
zip -r logs.zip logs \
    -x "*.tmp" "*.cache"
```

### 压缩当前目录所有文件

```bash
zip -r all.zip .
```

### 压缩多个指定目录

```bash
zip -r backup.zip data config scripts
```

---

## 与 tar.gz 对比

| 特性         | zip | tar.gz |
| ---------- | --- | ------ |
| Windows兼容性 | 很好  | 较差     |
| Linux使用频率  | 较高  | 更高     |
| 保留Linux权限  | 一般  | 完整     |
| 压缩率        | 一般  | 较高     |
| 适合跨平台传输    | 是   | 一般     |

在 Linux 服务器备份、软件发布场景中，更常见的是：

```bash
tar -zcvf backup.tar.gz directory/
```

而需要与 Windows 用户交换文件时，通常使用：

```bash
zip -r backup.zip directory/
```