---
title: Linux 文件系统修复
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4Ana3e
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/140'
---

# Linux 文件系统修复

<img src="images/Linux.svg" width="300">

Linux 文件系统修复的方法取决于**文件系统类型**、**故障原因**以及**系统是否还能启动**。下面按照实际运维中最常见的场景进行介绍。

---

# 一、常见故障现象

Linux 文件系统损坏通常表现为：

* 开机进入 Emergency Mode
* 出现 `fsck` 提示
* 日志中出现

  ```
  EXT4-fs error
  XFS (dm-0): Metadata corruption
  Buffer I/O error
  ```
* 文件无法读取
* 挂载失败

  ```
  mount: wrong fs type, bad option, bad superblock
  ```

首先建议查看日志：

```bash
dmesg | tail -100

journalctl -xb
```

---

# 二、先确认文件系统类型

查看磁盘

```bash
lsblk -f
```

例如

```
NAME   FSTYPE LABEL UUID
sda
├─sda1 ext4
├─sda2 xfs
└─sda3 swap
```

或者

```bash
blkid
```

不同文件系统修复方式不同。

常见包括：

| 文件系统           | 修复工具        |
| -------------- | ----------- |
| ext2/ext3/ext4 | fsck、e2fsck |
| xfs            | xfs_repair  |
| btrfs          | btrfs check |
| FAT            | fsck.vfat   |
| NTFS           | ntfsfix     |

---

# 三、修复 ext4 文件系统

## 1）卸载文件系统

不能修复正在使用的分区。

例如

```bash
umount /dev/sdb1
```

如果是根目录：

需要进入 Rescue Mode 或 LiveCD。

---

## 2）执行检查

推荐：

```bash
fsck -f /dev/sdb1
```

或者

```bash
e2fsck -f /dev/sdb1
```

参数说明：

```
-f    强制检查
-y    自动回答 yes
-p    自动修复简单错误
```

例如

```bash
e2fsck -fy /dev/sdb1
```

输出类似：

```
Pass 1: Checking inodes
Pass 2: Checking directory structure
Pass 3: Checking connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary

FILE SYSTEM WAS MODIFIED
```

说明已经修复。

---

# 四、修复 XFS

XFS **不能使用 fsck**。

查看

```bash
xfs_info /dev/sdb1
```

修复：

```bash
umount /dev/sdb1

xfs_repair /dev/sdb1
```

如果日志损坏：

```bash
xfs_repair -L /dev/sdb1
```

注意：

```
-L
```

表示删除日志。

可能丢失最近尚未写入的数据。

---

# 五、修复 Btrfs

检查：

```bash
btrfs check /dev/sdb1
```

如果需要修复：

```bash
btrfs check --repair /dev/sdb1
```

官方建议：

**不要轻易使用 `--repair`**。

很多情况下建议先：

```bash
mount -o recovery
```

或者恢复快照。

---

# 六、坏超级块（Superblock）修复

如果出现：

```
bad superblock
```

ext4 会保存多个备用 Superblock。

查看：

```bash
mke2fs -n /dev/sdb1
```

输出：

```
Superblock backups stored on blocks:

32768
98304
163840
229376
```

然后：

```bash
e2fsck -b 32768 /dev/sdb1
```

即可利用备用 Superblock 修复。

---

# 七、根文件系统修复

如果系统不能启动：

进入 GRUB。

进入

```
Recovery Mode
```

或者使用 LiveCD。

然后：

```bash
fsck /dev/sda2
```

修复完成：

```bash
reboot
```

---

# 八、LVM 环境

先激活 VG：

```bash
vgchange -ay
```

查看：

```bash
lvs
```

例如：

```
/dev/mapper/rhel-root
```

修复：

```bash
fsck /dev/mapper/rhel-root
```

或者：

```bash
xfs_repair /dev/mapper/rhel-root
```

---

# 九、RAID 环境

如果是软件 RAID：

查看状态：

```bash
cat /proc/mdstat
```

确认 RAID 正常：

```bash
mdadm --detail /dev/md0
```

不要直接修复某块磁盘，而应修复 RAID 逻辑设备，例如：

```bash
fsck /dev/md0
```

---

# 十、强制系统下次启动检查

对于 ext 文件系统：

```bash
touch /forcefsck
```

或者

```bash
shutdown -r now
```

启动时会自动执行：

```
fsck
```

---

# 十一、查看磁盘是否存在硬件故障

很多文件系统损坏其实来自硬盘问题。

安装 SMART 工具：

```bash
sudo apt install smartmontools
```

查看：

```bash
smartctl -a /dev/sda
```

重点关注：

```
Reallocated_Sector_Ct
Pending_Sector
Offline_Uncorrectable
```

如果这些指标持续增加，应尽快备份数据并更换硬盘。

---

# 十二、常用修复命令汇总

| 文件系统  | 检查命令                           | 修复命令                                 |
| ----- | ------------------------------ | ------------------------------------ |
| ext4  | `e2fsck -f /dev/sdb1`          | `e2fsck -fy /dev/sdb1`               |
| xfs   | `xfs_repair -n /dev/sdb1`（仅检查） | `xfs_repair /dev/sdb1`               |
| btrfs | `btrfs check /dev/sdb1`        | `btrfs check --repair /dev/sdb1`（谨慎） |
| FAT   | `fsck.vfat /dev/sdb1`          | `fsck.vfat -a /dev/sdb1`             |
| NTFS  | `ntfsfix /dev/sdb1`            | `ntfsfix /dev/sdb1`                  |

---

# 十三、企业运维中的最佳实践

1. **先备份数据**：如果磁盘仍可读取，优先使用 `rsync`、`tar` 等方式备份重要数据。
2. **确认硬件状态**：先检查 SMART 信息，避免在存在物理故障的磁盘上反复修复，导致数据进一步损坏。
3. **不要在线修复**：除支持在线检查的特殊场景外，应先卸载文件系统或进入救援模式，再执行修复。
4. **先检查后修复**：例如使用 `e2fsck -n` 或 `xfs_repair -n` 评估问题，再决定是否执行实际修复。
5. **保留日志**：修复前后保存 `dmesg`、`journalctl`、`smartctl` 输出，便于分析故障原因和追踪问题。
