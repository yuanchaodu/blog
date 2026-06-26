---
title: Linux mount 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnYua
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/132'
---

# Linux mount 命令

<img src="images/Linux.svg" width="300">

`mount` 是 Linux 中用来**挂载文件系统**的命令。简单说，就是把磁盘分区、U 盘、ISO 镜像、网络共享目录等，接入到 Linux 的某个目录下，让用户可以访问里面的文件。

## 一、基本语法

```bash
mount [选项] 设备 挂载点
```

例如：

```bash
mount /dev/sdb1 /mnt/usb
```

意思是：把 `/dev/sdb1` 这个分区挂载到 `/mnt/usb` 目录。

挂载后，访问 `/mnt/usb` 就是在访问这个分区里的内容。

## 二、常用查看命令

查看当前已挂载的文件系统：

```bash
mount
```

或：

```bash
findmnt
```

查看磁盘和分区：

```bash
lsblk
```

查看文件系统类型：

```bash
blkid
```

## 三、常用挂载示例

### 1. 挂载普通磁盘分区

```bash
mount /dev/sdb1 /data
```

如果目录不存在，先创建：

```bash
mkdir -p /data
```

### 2. 指定文件系统类型

```bash
mount -t ext4 /dev/sdb1 /data
```

常见类型有：

```text
ext4、xfs、ntfs、vfat、iso9660、nfs、cifs
```

### 3. 挂载 ISO 镜像

```bash
mount -o loop /root/software.iso /mnt/iso
```

### 4. 挂载为只读

```bash
mount -o ro /dev/sdb1 /mnt/test
```

### 5. 重新挂载为读写

```bash
mount -o remount,rw /data
```

### 6. 挂载 NFS 网络目录

```bash
mount -t nfs 192.168.1.10:/share /mnt/nfs
```

### 7. 挂载 Windows 共享目录

```bash
mount -t cifs //192.168.1.20/share /mnt/win -o username=user,password=pass
```

## 四、卸载文件系统

```bash
umount /data
```

或：

```bash
umount /dev/sdb1
```

注意命令是 `umount`，不是 `unmount`。

如果提示设备忙：

```bash
lsof +f -- /data
```

或：

```bash
fuser -vm /data
```

查看是谁正在占用。

## 五、开机自动挂载

编辑：

```bash
/etc/fstab
```

示例：

```text
UUID=xxxx-xxxx  /data  ext4  defaults  0  2
```

建议使用 `UUID`，不要直接写 `/dev/sdb1`，因为设备名可能变化。

测试配置是否正确：

```bash
mount -a
```

如果没有报错，说明配置基本正常。

## 六、常见参数

```text
-t    指定文件系统类型
-o    指定挂载选项
-a    挂载 /etc/fstab 中所有未挂载项
-r    只读挂载
-w    读写挂载
-v    显示详细信息
```

常见 `-o` 选项：

```text
ro        只读
rw        读写
noexec    禁止执行程序
nosuid    禁止 suid 权限
nodev     禁止设备文件生效
remount   重新挂载
loop      挂载镜像文件
defaults  默认参数
```

## 七、常见排错

### 1. 提示挂载点不存在

```text
mount point does not exist
```

处理：

```bash
mkdir -p /mnt/test
```

### 2. 提示文件系统类型错误

```text
wrong fs type
```

处理：

```bash
blkid /dev/sdb1
mount -t 正确类型 /dev/sdb1 /mnt/test
```

### 3. 提示设备忙

```text
target is busy
```

处理：

```bash
fuser -vm /mnt/test
```

### 4. `/etc/fstab` 配错导致启动异常

进入救援模式后修改 `/etc/fstab`，或先用：

```bash
mount -a
```

提前测试，避免重启后出问题。

## 八、实用流程

挂载一块新磁盘通常这样做：

```bash
lsblk
blkid
mkdir -p /data
mount /dev/sdb1 /data
df -h
```

需要永久挂载时：

```bash
blkid /dev/sdb1
vi /etc/fstab
mount -a
df -h
```

一句话总结：`mount` 就是把“存储设备或网络目录”接到 Linux 的某个目录上，接好后就像普通目录一样使用。
