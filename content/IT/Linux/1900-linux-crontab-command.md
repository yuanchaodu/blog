---
title: Linux crontab 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKrR
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/190'
---

<img src="images/Linux.svg" width="300">

# Linux crontab 命令

`crontab` 是 Linux 中用于**管理定时任务**的命令。它可以让系统按照指定的时间自动执行脚本或命令，例如每天备份数据库、定时清理日志、定时发送邮件等。

## 一、crontab 常用命令

| 命令                  | 说明                   |
| ------------------- | -------------------- |
| `crontab -e`        | 编辑当前用户的定时任务          |
| `crontab -l`        | 查看当前用户的定时任务          |
| `crontab -r`        | 删除当前用户的所有定时任务（谨慎使用）  |
| `crontab -i -r`     | 删除前进行确认              |
| `crontab -u 用户名 -l` | 查看指定用户的定时任务（需要 root） |
| `crontab -u 用户名 -e` | 编辑指定用户的定时任务（需要 root） |

例如：

```bash
crontab -l
```

查看当前用户所有定时任务。

---

## 二、crontab 文件格式

每一行表示一个定时任务：

```text
* * * * * command
│ │ │ │ │
│ │ │ │ └── 星期（0-7，0 和 7 都表示星期日）
│ │ │ └──── 月份（1-12）
│ │ └────── 日期（1-31）
│ └──────── 小时（0-23）
└────────── 分钟（0-59）
```

格式为：

```text
分钟 小时 日期 月份 星期 命令
```

例如：

```bash
30 2 * * * /home/user/backup.sh
```

表示：

> 每天凌晨 **2:30** 执行 `backup.sh`。

---

## 三、特殊符号

### 1. `*`

表示"任意值"。

例如：

```bash
* * * * *
```

表示每分钟执行一次。

---

### 2. `,`

表示多个值。

例如：

```bash
0 8,12,18 * * *
```

表示每天：

* 08:00
* 12:00
* 18:00

执行一次。

---

### 3. `-`

表示范围。

例如：

```bash
0 9-17 * * *
```

表示每天 **9 点到 17 点**每小时执行一次。

即：

```
09:00
10:00
11:00
...
17:00
```

---

### 4. `/`

表示步长。

例如：

```bash
*/5 * * * *
```

表示每 **5 分钟**执行一次。

等价于：

```bash
0,5,10,15,20...
```

再例如：

```bash
0 */2 * * *
```

表示每 **2 小时**执行一次。

---

## 四、常见示例

### 每分钟执行

```bash
* * * * * /home/test.sh
```

---

### 每5分钟执行

```bash
*/5 * * * * /home/test.sh
```

---

### 每小时执行

```bash
0 * * * * /home/test.sh
```

---

### 每天凌晨1点执行

```bash
0 1 * * * /home/test.sh
```

---

### 每周一凌晨2点

```bash
0 2 * * 1 /home/test.sh
```

---

### 每月1号凌晨3点

```bash
0 3 1 * * /home/test.sh
```

---

### 每年1月1日0点

```bash
0 0 1 1 * /home/test.sh
```

---

### 工作日每天8:30

```bash
30 8 * * 1-5
```

---

### 每隔10分钟执行

```bash
*/10 * * * *
```

---

### 每30秒执行（crontab 不支持）

`crontab` 最小时间粒度为 **1 分钟**，不能直接设置每 30 秒执行。

可采用如下方式：

```bash
* * * * * /home/test.sh
* * * * * sleep 30; /home/test.sh
```

或使用 `systemd timer`、常驻程序等更合适的方案。

---

## 五、环境变量

`cron` 运行时使用的是一个精简环境，因此建议在任务中使用**绝对路径**。

例如：

```bash
PATH=/usr/local/bin:/usr/bin:/bin

0 2 * * * /usr/bin/python3 /home/app/main.py
```

如果脚本依赖环境变量，可以在脚本中显式设置，或在 `crontab` 顶部定义：

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=""
```

---

## 六、日志输出

建议将标准输出和错误输出重定向到日志文件：

```bash
0 * * * * /home/test.sh >> /var/log/test.log 2>&1
```

其中：

* `>>`：追加标准输出。
* `2>&1`：将标准错误重定向到标准输出。

如果不需要任何输出：

```bash
0 * * * * /home/test.sh >/dev/null 2>&1
```

---

## 七、查看 Cron 服务状态

不同 Linux 发行版使用的服务名称可能不同。

### CentOS / RHEL

```bash
systemctl status crond
```

启动：

```bash
systemctl start crond
```

设置开机启动：

```bash
systemctl enable crond
```

---

### Ubuntu / Debian

```bash
systemctl status cron
```

启动：

```bash
sudo systemctl start cron
```

开机启动：

```bash
sudo systemctl enable cron
```

---

## 八、排查任务未执行的方法

如果定时任务没有按预期运行，可以依次检查：

1. **Cron 服务是否启动**

   ```bash
   systemctl status cron      # Ubuntu/Debian
   systemctl status crond     # CentOS/RHEL
   ```

2. **任务是否已正确保存**

   ```bash
   crontab -l
   ```

3. **命令是否使用绝对路径**

   ```bash
   /usr/bin/python3 /home/user/test.py
   ```

4. **脚本是否具有执行权限**

   ```bash
   chmod +x test.sh
   ```

5. **查看系统日志**

   * Ubuntu/Debian：

     ```bash
     grep CRON /var/log/syslog
     ```
   * CentOS/RHEL：

     ```bash
     grep CRON /var/log/cron
     ```

6. **检查脚本运行环境**
   在脚本中打印环境变量或显式设置 `PATH`、`HOME` 等，避免因环境差异导致任务失败。

## 九、速查表

| 表达式            | 含义                  |
| -------------- | ------------------- |
| `* * * * *`    | 每分钟执行               |
| `*/5 * * * *`  | 每 5 分钟执行            |
| `0 * * * *`    | 每小时整点执行             |
| `0 0 * * *`    | 每天 00:00 执行         |
| `30 8 * * 1-5` | 工作日 08:30 执行        |
| `0 2 * * 0`    | 每周日 02:00 执行        |
| `0 3 1 * *`    | 每月 1 日 03:00 执行     |
| `0 0 1 1 *`    | 每年 1 月 1 日 00:00 执行 |

对于现代 Linux 系统，如果需要比 `crontab` 更灵活的调度（如支持秒级、依赖关系、失败重试、按事件触发等），可以考虑使用 **systemd timer**。而对于传统的周期性任务，`crontab` 仍然是最简单、最广泛使用的方案。
