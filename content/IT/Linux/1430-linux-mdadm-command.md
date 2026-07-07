---
title: Linux mdadm 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnnX_
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/143'
---

# Linux mdadm 命令

<img src="images/Linux.svg" width="300">

`mdadm` 是 Linux 下管理 **软件 RAID** 的常用命令，用来创建、查看、维护磁盘阵列。

## 常用命令

查看 RAID 状态：

```bash
cat /proc/mdstat
```

查看某个阵列详情：

```bash
mdadm --detail /dev/md0
```

扫描已有 RAID：

```bash
mdadm --detail --scan
```

创建 RAID1：

```bash
mdadm --create /dev/md0 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb /dev/sdc
```

创建 RAID5：

```bash
mdadm --create /dev/md0 \
  --level=5 \
  --raid-devices=3 \
  /dev/sdb /dev/sdc /dev/sdd
```

停止阵列：

```bash
mdadm --stop /dev/md0
```

重新组装阵列：

```bash
mdadm --assemble /dev/md0 /dev/sdb /dev/sdc
```

自动扫描并组装：

```bash
mdadm --assemble --scan
```

添加新磁盘：

```bash
mdadm /dev/md0 --add /dev/sdd
```

标记磁盘故障：

```bash
mdadm /dev/md0 --fail /dev/sdb
```

移除故障磁盘：

```bash
mdadm /dev/md0 --remove /dev/sdb
```

查看磁盘 RAID 信息：

```bash
mdadm --examine /dev/sdb
```

## 常见维护流程

更换故障盘：

```bash
mdadm /dev/md0 --fail /dev/sdb
mdadm /dev/md0 --remove /dev/sdb
mdadm /dev/md0 --add /dev/sde
```

查看重建进度：

```bash
watch cat /proc/mdstat
```

保存配置：

```bash
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
```

更新 initramfs：

```bash
update-initramfs -u
```

在 RHEL/CentOS 系统中常用：

```bash
mdadm --detail --scan >> /etc/mdadm.conf
dracut -H -f
```

## 提醒

操作前建议先确认磁盘名：

```bash
lsblk
fdisk -l
blkid
```

`mdadm` 操作有风险，尤其是 `--create`、`--zero-superblock`、`--remove`，可能导致数据丢失。生产环境中建议先备份重要数据，再操作。
