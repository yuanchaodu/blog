---
title: Linux pstree 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSHU
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/96'
---

#  Linux pstree 命令

<img src="images/Linux.svg" width="300">

`pstree` 是 Linux 中一个非常实用的进程查看工具，它以**树状结构**显示系统中的进程及其父子关系，比 `ps` 命令更直观，特别适合分析进程启动链路、服务依赖关系以及排查僵尸进程。

## 一、基本语法

```bash
pstree [选项]
```

安装（部分系统默认未安装）：

### CentOS/RHEL

```bash
yum install psmisc
```

### Rocky Linux/AlmaLinux

```bash
dnf install psmisc
```

### Ubuntu/Debian

```bash
apt install psmisc
```

---

## 二、常用示例

### 查看所有进程树

```bash
pstree
```

示例输出：

```text
systemd─┬─NetworkManager
        ├─sshd───sshd───bash───pstree
        ├─cron
        └─rsyslogd
```

含义：

* `systemd` 是 PID 1 的根进程
* `sshd` 启动了登录会话
* `bash` 是当前 Shell
* `pstree` 是当前执行的命令

---

### 显示进程 PID

```bash
pstree -p
```

输出示例：

```text
systemd(1)─┬─NetworkManager(734)
           ├─sshd(1023)───sshd(2048)───bash(2050)───pstree(3056)
           └─cron(890)
```

适用于定位具体进程号。

---

### 显示指定用户的进程树

```bash
pstree username
```

例如：

```bash
pstree root
```

---

### 显示进程所属用户

```bash
pstree -u
```

示例：

```text
systemd(root)─┬─sshd(root)───sshd(nick)───bash(nick)
```

---

### 高亮当前进程及祖先进程

```bash
pstree -h
```

便于查看当前会话所在位置。

---

### 显示命令行参数

```bash
pstree -a
```

例如：

```text
sshd,1023
 └─sshd,2048
     └─bash
         └─python app.py
```

用于查看进程启动参数。

---

### 同时显示 PID 和参数

```bash
pstree -ap
```

示例：

```text
systemd(1)
 └─sshd(1023)
     └─bash(2050)
         └─python(3001) app.py
```

这是运维排障中最常用的组合之一。

---

## 三、指定 PID 查看进程树

查看某个进程及其子进程：

```bash
pstree -p 1234
```

例如：

```bash
pstree -p 1
```

输出：

```text
systemd(1)─┬─NetworkManager(734)
           ├─sshd(1023)
           └─dockerd(1500)
```

查看某个 Java 进程派生的所有子进程：

```bash
pstree -p <java_pid>
```

---

## 四、生产环境常见用途

### 1. 查看服务启动关系

例如查看 Nginx：

```bash
pstree -p | grep nginx
```

输出：

```text
nginx(1200)─┬─nginx(1201)
            ├─nginx(1202)
            └─nginx(1203)
```

说明：

* 1200 是 Master 进程
* 1201~1203 是 Worker 进程

---

### 2. 查看 SSH 登录链路

```bash
pstree -ap | grep sshd
```

输出：

```text
sshd(1023)
 └─sshd(2048)
     └─bash(2050)
         └─top(3001)
```

可以快速定位用户当前运行的命令。

---

### 3. 排查僵尸进程

```bash
pstree -p
```

若出现：

```text
systemd(1)
 └─bash(2000)
     └─python(3000)
         └─python(3001)
```

结合：

```bash
ps -ef | grep defunct
```

可以查找未被父进程回收的子进程。

---

### 4. 分析 Docker 进程关系

```bash
pstree -p | grep docker
```

可能看到：

```text
dockerd(1500)─┬─containerd(1600)
              └─containerd-shim(1700)
                  └─java(1800)
```

用于分析容器与宿主机进程的关系。

---

## 五、与 ps 命令对比

| 功能       | pstree | ps |
| -------- | ------ | -- |
| 查看进程列表   | √      | √  |
| 查看父子关系   | √√√    | 一般 |
| 查看 PID   | √      | √  |
| 查看资源占用   | ×      | √  |
| 查看完整参数   | √      | √  |
| 排查进程继承关系 | √√√    | 较弱 |

常见组合：

```bash
pstree -ap
```

查看进程关系。

```bash
ps -ef
```

查看进程详细信息。

```bash
top
```

查看资源消耗。

---

## 六、运维人员最常用的几个命令

```bash
pstree -p
```

显示 PID。

```bash
pstree -ap
```

显示 PID 和启动参数。

```bash
pstree -pu
```

显示 PID 和所属用户。

```bash
pstree -p <PID>
```

查看指定进程的子进程树。

```bash
pstree -s <PID>
```

显示指定进程的祖先进程链。

例如：

```bash
pstree -s 3001
```

输出：

```text
systemd(1)──sshd(1023)──bash(2050)──python(3001)
```

这对于分析“某个进程是谁启动的”特别有帮助。
