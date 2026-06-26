---
title: Linux fstab 配置
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnYw4
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/134'
---

# Linux fstab 配置

<img src="images/Linux.svg" width="300">

`/etc/fstab` 是 Linux 用来配置**开机自动挂载磁盘、分区、网络存储、交换分区**的文件。

## 1. 基本格式

每一行通常有 6 个字段：

```bash
<设备>  <挂载点>  <文件系统类型>  <挂载选项>  <dump>  <fsck>
```

示例：

```bash
UUID=xxxx-xxxx  /data  ext4  defaults  0  2
```

含义如下：

| 字段     | 含义                      |
| ------ | ----------------------- |
| 设备     | 磁盘分区，可以用 UUID、设备名、LABEL |
| 挂载点    | 挂载到哪个目录                 |
| 文件系统类型 | 如 ext4、xfs、nfs、swap     |
| 挂载选项   | 如 defaults、rw、noatime   |
| dump   | 备份标记，通常写 0              |
| fsck   | 开机磁盘检查顺序                |

## 2. 常见配置示例

### 挂载 ext4 分区

```bash
UUID=1234abcd-5678-ef00-1111-222233334444  /data  ext4  defaults  0  2
```

### 挂载 xfs 分区

```bash
UUID=1234abcd-5678-ef00-1111-222233334444  /data  xfs  defaults  0  0
```

### 挂载 swap

```bash
UUID=xxxx-xxxx  none  swap  sw  0  0
```

### 挂载 Windows NTFS 分区

```bash
UUID=xxxx-xxxx  /mnt/windows  ntfs-3g  defaults  0  0
```

### 挂载 NFS 网络共享

```bash
192.168.1.10:/share/data  /mnt/nfs  nfs  defaults,_netdev  0  0
```

## 3. 推荐用 UUID

查看 UUID：

```bash
blkid
```

或：

```bash
lsblk -f
```

不建议长期使用：

```bash
/dev/sdb1
```

因为磁盘顺序变化后，设备名可能变成 `/dev/sdc1`，容易挂错。

## 4. 常用挂载选项

| 选项       | 说明             |
| -------- | -------------- |
| defaults | 默认选项           |
| rw       | 可读写            |
| ro       | 只读             |
| noatime  | 不记录访问时间，减少磁盘写入 |
| auto     | 开机自动挂载         |
| noauto   | 不自动挂载          |
| user     | 允许普通用户挂载       |
| _netdev  | 网络设备，等网络可用后再挂载 |
| nofail   | 挂载失败不影响系统启动    |

比较安全的配置可以这样写：

```bash
UUID=xxxx  /data  ext4  defaults,nofail  0  2
```

## 5. 修改后的测试方法

修改 `/etc/fstab` 后，不要直接重启，先测试：

```bash
mount -a
```

如果没有报错，说明配置基本正确。

查看是否挂载成功：

```bash
df -h
```

或：

```bash
findmnt
```

## 6. 注意事项

修改前建议备份：

```bash
cp /etc/fstab /etc/fstab.bak
```

如果配置错误，可能导致系统启动异常。生产服务器上建议加入 `nofail`，尤其是非系统盘和网络盘。
