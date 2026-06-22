---
title: Linux pgrep 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnSBt
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/95'
---

#  Linux pgrep 命令

<img src="images/Linux.svg" width="300">

`pgrep`（Process Grep）是 Linux 中用于**根据进程名称、用户、启动参数等条件查找进程 PID** 的命令。相比 `ps -ef | grep`，它更简洁，也更适合在脚本中使用。

## 基本语法

```bash
pgrep [选项] 模式
```

其中：

* `模式`：通常是进程名或匹配进程信息的正则表达式。
* 返回结果为符合条件的进程 PID。

---

## 常用示例

### 1. 查找指定进程

查找 nginx 进程：

```bash
pgrep nginx
```

输出：

```bash
1234
1235
```

表示找到两个 nginx 进程。

---

### 2. 显示 PID 和进程名

```bash
pgrep -l nginx
```

输出：

```bash
1234 nginx
1235 nginx
```

参数说明：

* `-l`：显示 PID 和进程名称。

---

### 3. 显示完整命令行

```bash
pgrep -a java
```

输出：

```bash
4567 java -jar app.jar
```

参数说明：

* `-a`：显示完整启动命令。

---

### 4. 按用户查找进程

查找 root 用户运行的 sshd：

```bash
pgrep -u root sshd
```

---

### 5. 查找最新启动的进程

```bash
pgrep -n nginx
```

参数说明：

* `-n`（newest）：最新启动的匹配进程。

---

### 6. 查找最早启动的进程

```bash
pgrep -o nginx
```

参数说明：

* `-o`（oldest）：最早启动的匹配进程。

---

### 7. 精确匹配进程名

例如系统中有：

```bash
java
java_app
```

如果只查找 `java`：

```bash
pgrep -x java
```

参数说明：

* `-x`：精确匹配。

---

### 8. 根据完整命令匹配

默认只匹配进程名。

例如：

```bash
java -jar app.jar
```

查找包含 app.jar 的进程：

```bash
pgrep -f app.jar
```

参数说明：

* `-f`：匹配完整命令行。

---

### 9. 查找多个进程

```bash
pgrep "nginx|httpd"
```

输出：

```bash
1234
5678
```

支持正则表达式。

---

## 与 ps + grep 的比较

传统方式：

```bash
ps -ef | grep nginx
```

缺点：

```bash
root 1234 nginx
root 5678 grep nginx
```

会把 grep 自己也查出来。

使用 pgrep：

```bash
pgrep -l nginx
```

直接输出：

```bash
1234 nginx
```

更加简洁。

---

## 在脚本中的应用

### 判断进程是否存在

```bash
if pgrep nginx > /dev/null
then
    echo "nginx 正在运行"
else
    echo "nginx 未运行"
fi
```

---

### 获取 PID

```bash
PID=$(pgrep -x nginx)

echo $PID
```

---

### 配合 kill 使用

终止所有 nginx 进程：

```bash
kill $(pgrep nginx)
```

或者直接使用：

```bash
pkill nginx
```

其中：

* `pgrep`：查找进程。
* `pkill`：查找并发送信号给进程。

---

## 常用参数汇总

| 参数        | 说明          |
| --------- | ----------- |
| `-l`      | 显示 PID 和进程名 |
| `-a`      | 显示完整命令行     |
| `-f`      | 匹配完整命令行     |
| `-x`      | 精确匹配进程名     |
| `-u user` | 按用户查找       |
| `-U uid`  | 按用户ID查找     |
| `-P pid`  | 查找指定父进程的子进程 |
| `-n`      | 最新启动的进程     |
| `-o`      | 最早启动的进程     |
| `-c`      | 仅返回匹配数量     |
| `-v`      | 反向匹配        |

---

## 企业运维中的常见用法

### 查看 Java 应用是否运行

```bash
pgrep -af java
```

输出：

```bash
4567 java -Xms2g -Xmx4g -jar mes.jar
```

---

### 统计某类进程数量

```bash
pgrep -c python
```

输出：

```bash
12
```

表示当前有 12 个 Python 进程。

---

### 查找某个服务的所有子进程

假设主进程 PID 为 1000：

```bash
pgrep -P 1000
```

---

### 查找包含关键参数的进程

例如查找连接 Oracle 数据库的程序：

```bash
pgrep -af oracle
```

或者：

```bash
pgrep -f jdbc:oracle
```

---

## 查看帮助

```bash
man pgrep
```

或：

```bash
pgrep --help
```

`pgrep` 和 `pkill` 都属于 Linux 的 **procps/procps-ng** 工具集，是日常运维、Shell 脚本和服务监控中非常常用的进程管理工具。对于生产环境中的 Java、Nginx、Oracle、Docker 等服务，推荐优先使用 `pgrep -af` 来定位进程，比 `ps | grep` 更简洁可靠。
