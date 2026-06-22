---
title: Linux killall 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSOP
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/98'
---

#  Linux killall 命令

<img src="images/Linux.svg" width="300">

`killall` 是 Linux 中用于**按照进程名终止进程**的命令。与 `kill` 需要指定 PID（进程号）不同，`killall` 可以直接根据进程名称结束一个或多个进程。

## 基本语法

```bash
killall [选项] 进程名
```

例如：

```bash
killall firefox
```

表示终止所有名为 `firefox` 的进程。

---

## 常用用法

### 1. 结束指定进程

```bash
killall nginx
```

结束所有 `nginx` 进程。

---

### 2. 强制结束进程

默认发送的是 `SIGTERM`（15）信号，允许程序正常退出。

如果程序无响应，可以发送 `SIGKILL`（9）：

```bash
killall -9 nginx
```

或者：

```bash
killall -SIGKILL nginx
```

---

### 3. 结束指定用户的进程

```bash
killall -u oracle
```

结束用户 `oracle` 的所有进程。

也可以指定用户和进程：

```bash
killall -u oracle java
```

结束用户 `oracle` 启动的所有 `java` 进程。

---

### 4. 忽略大小写

```bash
killall -I Java
```

匹配 `java`、`Java`、`JAVA` 等名称。

---

### 5. 交互确认

执行前逐个确认：

```bash
killall -i nginx
```

输出类似：

```text
Kill nginx(1234) ? (y/N)
```

---

### 6. 查看支持的信号

```bash
killall -l
```

输出：

```text
HUP INT QUIT ILL ABRT FPE KILL SEGV PIPE ALRM TERM ...
```

---

## 查看进程后再终止

通常建议先查看进程：

```bash
ps -ef | grep nginx
```

或者：

```bash
pgrep nginx
```

确认后再执行：

```bash
killall nginx
```

---

## kill 与 killall 的区别

| 命令      | 根据什么终止  | 示例              |
| ------- | ------- | --------------- |
| kill    | PID     | `kill 1234`     |
| killall | 进程名     | `killall nginx` |
| pkill   | 名称或条件匹配 | `pkill nginx`   |

例如：

```bash
kill 1234
```

只结束 PID 为 1234 的进程。

```bash
killall nginx
```

结束所有名为 nginx 的进程。

```bash
pkill -f nginx
```

结束命令行中包含 nginx 的进程。

---

## 注意事项

### 1. 不同 Linux 发行版行为可能略有差异

在大多数 Linux 系统（如 Ubuntu、CentOS、Rocky Linux）中：

```bash
killall nginx
```

表示按进程名结束进程。

但某些 Unix 系统（如部分旧版 Unix）中，`killall` 可能会终止所有可终止进程，因此在跨平台环境中需特别谨慎。

### 2. 使用 `-9` 要慎重

```bash
killall -9 mysql
```

会直接杀死进程，可能导致：

* 数据未保存
* 文件损坏
* 数据库异常恢复

一般建议：

```bash
killall mysql
```

无效时再使用：

```bash
killall -9 mysql
```

---

## 实际运维示例

重启 Nginx：

```bash
killall nginx
nginx
```

结束所有 Java 应用：

```bash
killall java
```

结束所有 Python 脚本：

```bash
killall python3
```

结束某个用户启动的全部进程：

```bash
killall -u testuser
```

对于生产环境，更推荐先使用：

```bash
pgrep -a nginx
```

确认目标进程后再执行 `killall`，避免误杀业务进程。
