---
title: Linux Shell 项目实战
section: IT
category: Linux
---

<img src="images/Linux.svg" width="300">

# Linux Shell 项目实战

“Linux Shell 项目实战”可以从真实运维场景入手，边学命令，边完成可用脚本。建议按下面路线学习。

## 一、基础准备

先掌握这些核心内容：

* 文件与目录操作：`ls`、`cd`、`cp`、`mv`、`rm`、`find`
* 文本处理：`grep`、`sed`、`awk`、`cut`、`sort`
* 管道与重定向：`|`、`>`、`>>`、`2>`
* Shell变量、参数和数组
* 条件判断：`if`、`case`
* 循环：`for`、`while`
* 函数与返回值
* 定时任务：`cron`
* 日志记录和异常处理

## 二、推荐实战项目

### 项目1：服务器资源巡检脚本

定期检查：

* CPU使用率
* 内存使用率
* 磁盘空间
* 系统负载
* 运行时间
* 关键进程状态

适合学习 `top`、`free`、`df`、`uptime`、`ps`、`awk`。

### 项目2：日志分析脚本

分析系统或应用日志，统计：

* 错误数量
* 高频错误信息
* 访问量最高的IP
* HTTP状态码分布
* 某时间段内的异常记录

适合学习 `grep`、`awk`、`sort`、`uniq`、`sed`。

### 项目3：自动备份脚本

实现：

* 指定目录压缩备份
* 自动生成日期文件名
* 保留最近若干天备份
* 记录备份日志
* 备份失败时退出并报警

适合学习 `tar`、`date`、`find`、函数和退出码。

### 项目4：应用服务启停脚本

统一实现：

```bash
./app.sh start
./app.sh stop
./app.sh restart
./app.sh status
```

适合学习 `case`、进程管理、PID文件和函数封装。

### 项目5：批量服务器巡检

通过SSH连接多台服务器，执行相同检查，并生成汇总报告。

适合学习：

* `ssh`
* 循环
* 配置文件读取
* 超时控制
* 并发执行
* 结果汇总

## 三、示例：磁盘空间巡检脚本

```bash
#!/bin/bash

# 磁盘使用率报警阈值
THRESHOLD=80

# 日志文件
LOG_FILE="/tmp/disk_check.log"

echo "===== 磁盘巡检开始：$(date '+%F %T') =====" >> "$LOG_FILE"

df -P | awk 'NR > 1 {print $5, $6}' | while read -r usage mount_point
do
    usage_value=${usage%\%}

    if [ "$usage_value" -ge "$THRESHOLD" ]; then
        echo "警告：挂载点 $mount_point 使用率为 ${usage_value}%" \
            | tee -a "$LOG_FILE"
    else
        echo "正常：挂载点 $mount_point 使用率为 ${usage_value}%" \
            >> "$LOG_FILE"
    fi
done

echo "===== 磁盘巡检结束 =====" >> "$LOG_FILE"
```

运行方法：

```bash
chmod +x disk_check.sh
./disk_check.sh
```

## 四、推荐综合项目

结合企业运维场景，可以完成一个“Linux服务器自动巡检与报告系统”。

系统包含：

1. 读取服务器清单。
2. 远程采集CPU、内存、磁盘和进程信息。
3. 根据阈值判断是否异常。
4. 保存每台服务器的巡检结果。
5. 生成HTML或CSV汇总报告。
6. 通过定时任务每天自动执行。
7. 发现异常后发送邮件或企业消息。

这个项目覆盖Shell脚本的大部分常用知识，也适合直接应用到日常系统维护工作中。