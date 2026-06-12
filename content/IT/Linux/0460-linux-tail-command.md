---
title: Linux tail 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnE7u
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/46'
---

# Linux tail 命令

<img src="images/Linux.svg" width="300">

`tail` 是 Linux 中用于**查看文件末尾内容**的命令，常用于查看日志文件、监控程序运行状态等。

## 基本语法

```bash
tail [选项] 文件名
```

默认显示文件最后 10 行：

```bash
tail /var/log/messages
```

---

## 常用用法

### 1. 查看最后 N 行

查看最后 20 行：

```bash
tail -n 20 app.log
```

也可以简写：

```bash
tail -20 app.log
```

---

### 2. 实时监控文件变化

这是最常用的功能，特别适合查看日志。

```bash
tail -f app.log
```

当文件新增内容时，会自动显示出来。

例如：

```bash
tail -f /var/log/syslog
```

或

```bash
tail -f /var/log/messages
```

退出监控：

```bash
Ctrl + C
```

---

### 3. 实时监控多个文件

```bash
tail -f app.log error.log
```

输出示例：

```text
==> app.log <==
应用启动成功

==> error.log <==
数据库连接失败
```

---

### 4. 从指定行开始显示

从第 100 行开始显示到文件末尾：

```bash
tail -n +100 app.log
```

说明：

```text
+n  表示从第 n 行开始
-n  表示显示最后 n 行
```

例如：

```bash
tail -n +1 app.log
```

相当于显示整个文件。

---

### 5. 配合 grep 过滤内容

实时查看日志中的错误信息：

```bash
tail -f app.log | grep ERROR
```

忽略大小写：

```bash
tail -f app.log | grep -i error
```

---

### 6. 查看最后 N 个字节

显示最后 100 个字节：

```bash
tail -c 100 file.txt
```

显示最后 1KB：

```bash
tail -c 1024 file.txt
```

---

## 日志分析常见场景

### 查看最新错误日志

```bash
tail -100 error.log
```

---

### 实时查看 Tomcat 日志

```bash
tail -f catalina.out
```

---

### 查看 Nginx 访问日志

```bash
tail -f /var/log/nginx/access.log
```

---

### 查看 Nginx 错误日志

```bash
tail -f /var/log/nginx/error.log
```

---

### 查看 Docker 容器日志

虽然 Docker 提供专门命令，但效果类似：

```bash
docker logs -f 容器名
```

例如：

```bash
docker logs -f nginx
```

---

## 与 less 配合使用

查看最后 100 行并分页浏览：

```bash
tail -100 app.log | less
```

---

## 高级用法：处理日志轮转

普通 `-f` 在日志文件被重命名后可能失效。

推荐使用：

```bash
tail -F app.log
```

`-F` 等价于：

```bash
tail --follow=name --retry app.log
```

适用于：

```text
logrotate
Tomcat日志切割
Nginx日志切割
```

即使日志文件被重新创建，仍能继续跟踪。

---

## 与 head 的区别

| 命令   | 功能     |
| ---- | ------ |
| head | 查看文件开头 |
| tail | 查看文件末尾 |

例如：

```bash
head -20 app.log
```

查看前 20 行。

```bash
tail -20 app.log
```

查看后 20 行。

---

## 运维中最常用的几个命令

```bash
# 查看最后100行
tail -100 app.log

# 实时查看日志
tail -f app.log

# 支持日志轮转
tail -F app.log

# 实时查看错误
tail -f app.log | grep ERROR

# 查看最后1000行并搜索
tail -1000 app.log | grep "Exception"
```

对于日常服务器运维、应用排障和日志分析，`tail -f` 和 `tail -F` 是使用频率最高的两个用法。
