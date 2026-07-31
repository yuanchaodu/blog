---
title: Linux at 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKsB
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/191'
---

<img src="images/Linux.svg" width="300">

# Linux at 命令

## Linux `at` 命令

`at` 用于**安排一次性任务**，让系统在指定时间自动执行命令。

它和 `cron` 的区别是：

* `at`：执行一次。
* `cron`：按固定周期重复执行。

### 1. 基本用法

```bash
at 时间
```

进入交互界面后，输入要执行的命令。完成后按：

```text
Ctrl+D
```

例如，今天晚上 10 点执行脚本：

```bash
at 22:00
```

然后输入：

```bash
/home/nick/backup.sh
```

按 `Ctrl+D` 保存任务。

---

### 2. 常用时间格式

```bash
at 18:30
```

今天 18:30 执行。

```bash
at 10:00 tomorrow
```

明天 10:00 执行。

```bash
at now + 10 minutes
```

10 分钟后执行。

```bash
at now + 2 hours
```

2 小时后执行。

```bash
at 9:00 AM next Monday
```

下周一上午 9 点执行。

```bash
at midnight
```

当天或下一个午夜执行。

---

### 3. 直接提交命令

可以使用管道，避免进入交互界面：

```bash
echo "/home/nick/backup.sh" | at 22:00
```

例如，30 分钟后重启某个服务：

```bash
echo "systemctl restart nginx" | at now + 30 minutes
```

执行系统管理命令时，要注意当前用户是否有足够权限。

---

### 4. 查看待执行任务

```bash
atq
```

也可以使用：

```bash
at -l
```

示例输出：

```text
3   Fri Jul 31 22:00:00 2026 a nick
```

其中 `3` 是任务编号。

---

### 5. 查看任务内容

```bash
at -c 3
```

这里的 `3` 是任务编号。

输出中会包含环境变量和实际执行的命令，内容可能比较长。

---

### 6. 删除任务

```bash
atrm 3
```

也可以使用：

```bash
at -d 3
```

---

### 7. 安装并启动服务

部分 Linux 系统默认没有安装 `at`。

#### Ubuntu、Debian

```bash
sudo apt install at
sudo systemctl enable --now atd
```

#### CentOS、Rocky Linux、AlmaLinux、RHEL

```bash
sudo dnf install at
sudo systemctl enable --now atd
```

检查服务状态：

```bash
systemctl status atd
```

---

### 8. 输出重定向

`at` 任务没有普通终端窗口。建议把输出写入日志文件：

```bash
echo "/home/nick/backup.sh >> /var/log/backup.log 2>&1" | at 22:00
```

其中：

```text
>> /var/log/backup.log
```

表示追加保存标准输出。

```text
2>&1
```

表示把错误信息也写入同一个日志。

---

### 9. 注意事项

`at` 会继承提交任务时的大部分环境变量，但执行任务时的环境可能与登录终端不同。因此，脚本中最好使用完整路径：

```bash
/usr/bin/python3 /home/nick/scripts/report.py
```

不要只写：

```bash
python3 report.py
```

同时，尽量在脚本中明确工作目录：

```bash
cd /home/nick/scripts && /usr/bin/python3 report.py
```

一个较完整的例子：

```bash
echo "cd /home/nick/scripts && /usr/bin/python3 report.py >> /home/nick/logs/report.log 2>&1" | at 23:30
```

这条命令会在晚上 23:30 进入指定目录，运行 Python 程序，并将结果写入日志。
