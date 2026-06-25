---
title: Linux blkid 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnXTs
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/128'
---

# Linux blkid 命令

<img src="images/Linux.svg" width="300">

`blkid` 是 Linux 中用于**查看块设备（Block Device）属性**的命令。它可以读取磁盘或分区中的元数据，显示文件系统类型、UUID、LABEL、PARTUUID 等信息，因此常用于系统管理、磁盘挂载以及编写 `/etc/fstab`。

---

## 一、基本语法

```bash
blkid [选项] [设备]
```

例如：

```bash
blkid
```

输出类似：

```text
/dev/sda1: UUID="B2F6-1A3E" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI System Partition" PARTUUID="1d2f4d5a-01"
/dev/sda2: UUID="1e7c89f4-ec44-4b42-bd54-8cb2a7e78b62" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="1d2f4d5a-02"
/dev/sdb1: LABEL="DATA" UUID="89F6B1C1F6B18F5C" TYPE="ntfs"
```

---

# 二、常见用途

## 1. 查看所有磁盘和分区信息

```bash
blkid
```

输出所有已经识别文件系统的块设备。

例如：

```text
/dev/sda1: UUID="xxxx" TYPE="vfat"
/dev/sda2: UUID="yyyy" TYPE="ext4"
/dev/sdb1: UUID="zzzz" TYPE="xfs"
```

---

## 2. 查看指定设备

例如：

```bash
blkid /dev/sda2
```

输出：

```text
/dev/sda2: UUID="1e7c89..." TYPE="ext4"
```

---

## 3. 查看 UUID

很多时候只需要 UUID：

```bash
blkid -s UUID /dev/sda2
```

输出：

```text
UUID="1e7c89f4-ec44-4b42-bd54-8cb2a7e78b62"
```

如果只需要值：

```bash
blkid -s UUID -o value /dev/sda2
```

输出：

```text
1e7c89f4-ec44-4b42-bd54-8cb2a7e78b62
```

这是脚本中最常见的写法。

---

## 4. 查看文件系统类型

```bash
blkid -s TYPE /dev/sda2
```

输出：

```text
TYPE="ext4"
```

或者

```bash
blkid -s TYPE -o value /dev/sda2
```

输出：

```text
ext4
```

---

## 5. 查看 LABEL

```bash
blkid -s LABEL
```

输出：

```text
/dev/sdb1: LABEL="DATA"
```

---

## 6. 查看 PARTUUID

```bash
blkid -s PARTUUID
```

输出：

```text
/dev/sda2: PARTUUID="1d2f4d5a-02"
```

---

# 三、常用参数

| 参数   | 作用            |
| ---- | ------------- |
| `-s` | 指定输出某个字段      |
| `-o` | 指定输出格式        |
| `-L` | 根据 LABEL 查找设备 |
| `-U` | 根据 UUID 查找设备  |
| `-t` | 按条件搜索         |
| `-c` | 指定缓存文件        |
| `-p` | 直接探测设备，不依赖缓存  |
| `-l` | 列出符合条件的设备     |

---

# 四、输出格式

## 默认格式

```bash
blkid
```

输出：

```text
/dev/sda2: UUID="..." TYPE="ext4"
```

---

## value

```bash
blkid -o value -s UUID /dev/sda2
```

输出：

```text
1e7c89f4-ec44...
```

---

## export

```bash
blkid -o export /dev/sda2
```

输出：

```text
DEVNAME=/dev/sda2
UUID=1e7c89...
TYPE=ext4
PARTUUID=...
```

适合 Shell 脚本解析。

---

## full

```bash
blkid -o full
```

输出完整属性。

---

# 五、按条件搜索

例如寻找所有 ext4：

```bash
blkid -t TYPE=ext4
```

输出：

```text
/dev/sda2: UUID="..." TYPE="ext4"
```

寻找 UUID：

```bash
blkid -U 1e7c89f4-ec44-4b42-bd54-8cb2a7e78b62
```

输出：

```text
/dev/sda2
```

寻找 LABEL：

```bash
blkid -L DATA
```

输出：

```text
/dev/sdb1
```

---

# 六、在 `/etc/fstab` 中的应用

例如：

```text
UUID=1e7c89f4-ec44-4b42-bd54-8cb2a7e78b62 /data ext4 defaults 0 2
```

查看 UUID：

```bash
blkid /dev/sdb1
```

得到：

```text
UUID="1e7c89..."
```

然后写入 `fstab`。

这样即使磁盘名称从 `/dev/sdb1` 变成 `/dev/sdc1`，仍然能够正确挂载。

---

# 七、与其他命令的区别

| 命令         | 作用                     | 是否读取文件系统元数据     |
| ---------- | ---------------------- | --------------- |
| `blkid`    | 查看 UUID、LABEL、TYPE 等信息 | 是               |
| `lsblk`    | 查看磁盘树状结构、容量、挂载点        | 部分（可显示部分文件系统信息） |
| `fdisk -l` | 查看分区表                  | 否               |
| `df -h`    | 查看已挂载文件系统空间使用情况        | 否               |
| `mount`    | 查看当前挂载信息               | 否               |

例如：

```bash
lsblk -f
```

输出：

```text
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 vfat         B2F6-1A3E                            /boot/efi
└─sda2 ext4         1e7c89f4-ec44-4b42-bd54-8cb2a7e78b62 /
```

相比之下：

```bash
blkid
```

输出：

```text
/dev/sda1: UUID="B2F6-1A3E" TYPE="vfat"
/dev/sda2: UUID="1e7c89..." TYPE="ext4"
```

`lsblk -f` 更适合查看磁盘整体结构和挂载关系；`blkid` 更适合获取文件系统标识信息，尤其是在脚本和配置文件（如 `/etc/fstab`）中使用。

---

# 八、常见问题

### 1. 为什么有些分区没有输出？

`blkid` 默认只显示**能够识别文件系统或 RAID/LVM 签名**的块设备。如果某个分区尚未格式化，或没有可识别的元数据，它可能不会显示。

可以结合以下命令查看所有块设备：

```bash
lsblk
```

---

### 2. 为什么需要 root 权限？

普通用户通常可以读取大多数块设备的元数据，但某些设备可能因权限限制无法访问。遇到这种情况，可使用：

```bash
sudo blkid
```

---

### 3. `blkid` 信息是否会缓存？

是的，`blkid` 默认可能使用缓存（通常位于 `/run/blkid/blkid.tab` 或 `/etc/blkid.tab`，具体位置因发行版而异）。如果刚刚重新格式化了分区，缓存可能尚未更新。

此时可以直接探测设备：

```bash
sudo blkid -p /dev/sdb1
```

或删除缓存后重新执行 `blkid`。

---

## 总结

* **查看所有块设备信息**：`blkid`
* **查看指定设备**：`blkid /dev/sdXN`
* **获取 UUID**：`blkid -s UUID -o value /dev/sdXN`
* **获取文件系统类型**：`blkid -s TYPE -o value /dev/sdXN`
* **按类型查找**：`blkid -t TYPE=ext4`
* **按 UUID 查找设备**：`blkid -U <UUID>`
* **按 LABEL 查找设备**：`blkid -L <LABEL>`

在日常系统管理中，`blkid` 与 `lsblk -f` 往往配合使用：前者用于获取精确的文件系统标识（UUID、LABEL、TYPE 等），后者用于查看磁盘、分区及挂载关系。
