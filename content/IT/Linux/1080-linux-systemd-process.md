---
title: Linux systemd 进程管理
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSw9
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/108'
---

# Linux systemd 进程管理

<img src="images/Linux.svg" width="300">

Linux 中的 **systemd** 是当前大多数发行版默认的初始化系统（Init System）和服务管理框架，负责系统启动、服务管理、进程监控、日志管理等工作。

## 一、systemd 与进程管理的关系

传统 Linux 使用 `SysVinit` 管理系统服务，而现代 Linux 普遍采用 systemd。

可以把 systemd 理解为：

> Linux 系统中的“总调度员”，负责启动、停止、监控和重启各种后台进程（服务）。

例如：

* Web 服务（Nginx）
* 数据库服务（MySQL）
* SSH 服务
* Docker 服务

都可以由 systemd 统一管理。

---

## 二、核心概念

### 1. Unit（单元）

systemd 管理的对象称为 Unit。

常见类型：

| 类型      | 说明    |
| ------- | ----- |
| service | 服务进程  |
| socket  | 网络套接字 |
| target  | 运行级别  |
| mount   | 挂载点   |
| timer   | 定时任务  |
| path    | 文件监控  |

查看所有 Unit：

```bash
systemctl list-units
```

查看所有 Service：

```bash
systemctl list-units --type=service
```

---

### 2. Service（服务）

服务对应一个或多个进程。

例如：

```bash
sshd.service
```

对应 SSH 服务进程。

---

## 三、常用进程管理命令

### 查看服务状态

```bash
systemctl status sshd
```

输出内容包括：

* 主进程 PID
* 运行状态
* 启动时间
* 日志信息

例如：

```text
Active: active (running)
Main PID: 1234
```

---

### 启动服务

```bash
systemctl start nginx
```

---

### 停止服务

```bash
systemctl stop nginx
```

---

### 重启服务

```bash
systemctl restart nginx
```

---

### 重新加载配置

不重启进程：

```bash
systemctl reload nginx
```

---

### 查看是否运行

```bash
systemctl is-active nginx
```

结果：

```text
active
```

或：

```text
inactive
```

---

## 四、查看服务对应进程

查看 PID：

```bash
systemctl show nginx -p MainPID
```

输出：

```text
MainPID=1234
```

或者：

```bash
systemctl status nginx
```

---

查看进程树：

```bash
systemd-cgls
```

示例：

```text
system.slice
 └─nginx.service
     ├─1234 nginx
     └─1235 nginx
```

这可以清晰看到服务与子进程关系。

---

## 五、开机自启动管理

### 设置开机启动

```bash
systemctl enable nginx
```

---

### 取消开机启动

```bash
systemctl disable nginx
```

---

### 查看是否开机启动

```bash
systemctl is-enabled nginx
```

---

## 六、进程异常自动恢复

这是 systemd 很重要的能力。

编辑服务文件：

```ini
[Service]
ExecStart=/opt/app/start.sh

Restart=always
RestartSec=5
```

含义：

* 进程退出后自动重启
* 5 秒后重启

常见策略：

| 参数          | 说明     |
| ----------- | ------ |
| no          | 不重启    |
| on-failure  | 异常退出重启 |
| always      | 总是重启   |
| on-abnormal | 异常终止重启 |

适合：

* Java 服务
* Python 服务
* Node.js 服务
* 自研应用

---

## 七、systemd 与 cgroup

systemd 底层大量使用 Linux 的 cgroups 技术。

作用：

* 限制 CPU
* 限制内存
* 限制 I/O
* 管理进程组

例如：

```ini
[Service]
MemoryMax=2G
CPUQuota=50%
```

表示：

* 最大使用 2GB 内存
* 最多占用 50% CPU

---

## 八、日志管理

systemd 配套日志系统：

systemd-journald

查看服务日志：

```bash
journalctl -u nginx
```

实时查看：

```bash
journalctl -u nginx -f
```

查看本次启动日志：

```bash
journalctl -b
```

---

## 九、自定义服务管理

例如管理一个 Java 应用：

创建：

```bash
/etc/systemd/system/myapp.service
```

内容：

```ini
[Unit]
Description=My Java Application

[Service]
User=appuser
WorkingDirectory=/opt/myapp

ExecStart=/usr/bin/java -jar app.jar

Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

加载配置：

```bash
systemctl daemon-reload
```

启动：

```bash
systemctl start myapp
```

设置开机启动：

```bash
systemctl enable myapp
```

---

## 十、企业运维中最常用命令

```bash
# 查看服务状态
systemctl status xxx

# 启动服务
systemctl start xxx

# 停止服务
systemctl stop xxx

# 重启服务
systemctl restart xxx

# 开机启动
systemctl enable xxx

# 查看日志
journalctl -u xxx -f

# 查看PID
systemctl show xxx -p MainPID

# 查看全部失败服务
systemctl --failed

# 查看进程树
systemd-cgls
```

## 总结

systemd 的进程管理核心能力可以概括为：

1. **统一管理服务生命周期**（启动、停止、重启）。
2. **自动监控和恢复进程**。
3. **通过 cgroup 管理资源使用**。
4. **统一收集和查询日志**。
5. **支持开机自启动和依赖管理**。

对于企业服务器、数字化工厂平台、中间件、数据库以及自研应用部署来说，systemd 已经是 Linux 环境下最标准的进程管理方案。
