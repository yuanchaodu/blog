---
title: Linux shell 自动化脚本
section: IT
category: Linux
---

# Linux shell 自动化脚本

<img src="images/Linux.svg" width="300">

Linux Shell 自动化脚本，是指使用 Bash、Shell 等脚本语言，把重复的 Linux 操作自动执行。

常见用途包括：

* 自动备份文件、数据库和配置
* 批量创建、修改或删除文件
* 定时检查服务器、磁盘、内存和进程
* 自动部署应用程序
* 批量处理日志
* 自动启动、停止和重启服务
* 监控异常并发送告警

一个简单示例：

```bash
#!/bin/bash

# 定义备份目录
SOURCE_DIR="/data"
BACKUP_DIR="/backup"
DATE=$(date "+%Y%m%d_%H%M%S")

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 压缩备份
tar -czf "$BACKUP_DIR/data_$DATE.tar.gz" "$SOURCE_DIR"

# 删除30天前的备份
find "$BACKUP_DIR" -type f -name "*.tar.gz" -mtime +30 -delete

echo "备份完成：$BACKUP_DIR/data_$DATE.tar.gz"
```

保存为 `backup.sh` 后执行：

```bash
chmod +x backup.sh
./backup.sh
```

也可以加入 `crontab`，每天凌晨两点自动运行：

```bash
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

编写正式自动化脚本时，通常需要加入错误处理、日志记录、参数检查和执行结果判断，例如：

```bash
#!/bin/bash

set -euo pipefail

LOG_FILE="/var/log/my_script.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" | tee -a "$LOG_FILE"
}

main() {
    log "脚本开始执行"

    if ! systemctl is-active --quiet nginx; then
        log "检测到 nginx 未运行，正在启动"
        systemctl start nginx
    else
        log "nginx 运行正常"
    fi

    log "脚本执行完成"
}

main "$@"
```