---
title: Linux umount 命令
section: IT
category: Linux
---

# Linux umount 命令

<img src="images/Linux.svg" width="300">

`umount` 是 Linux 中用于**卸载（Unmount）已挂载文件系统**的命令。它与 `mount` 命令相对应，用于断开文件系统与挂载点之间的关联。

---

## 一、基本语法

```bash
umount [选项] <挂载点>
```

或者

```bash
umount [选项] <设备名>
```

例如：

```bash
umount /mnt/data
```

或者

```bash
umount /dev/sdb1
```

两种方式效果相同，只要指定的是当前已挂载的文件系统即可。

---

## 二、常见用法

### 1. 卸载指定挂载点

```bash
umount /mnt/usb
```

表示卸载挂载在 `/mnt/usb` 的文件系统。

---

### 2. 根据设备名卸载

```bash
umount /dev/sdb1
```

表示卸载设备 `/dev/sdb1`。

---

### 3. 卸载所有文件系统

```bash
umount -a
```

会按照 `/etc/mtab` 或 `/proc/mounts` 中记录卸载所有文件系统。

通常会配合使用：

```bash
umount -a -t nfs
```

只卸载 NFS 文件系统。

或者

```bash
umount -a -t ext4
```

只卸载 ext4 文件系统。

---

## 三、常用参数

| 参数      | 作用                 |
| ------- | ------------------ |
| `-a`    | 卸载所有挂载的文件系统        |
| `-t 类型` | 指定文件系统类型           |
| `-l`    | 延迟卸载（Lazy Unmount） |
| `-f`    | 强制卸载（主要用于 NFS）     |
| `-r`    | 如果卸载失败，则重新以只读方式挂载  |
| `-v`    | 显示详细过程             |

例如：

```bash
umount -v /mnt/data
```

输出类似：

```
/mnt/data umounted
```

---

## 四、为什么会卸载失败

最常见错误：

```text
umount: /mnt/data: target is busy
```

意思是：

> 挂载点正在被使用（Device or resource busy）。

例如：

* 当前 Shell 正位于该目录

```bash
cd /mnt/data
```

然后执行

```bash
umount /mnt/data
```

会失败。

---

还有可能是：

* 有程序正在访问文件
* 有终端停留在目录下
* 数据库仍在运行
* 日志文件正在写入

---

## 五、查看是谁占用了挂载点

### 方法一：`lsof`

```bash
lsof +D /mnt/data
```

显示正在访问该目录的进程。

例如：

```
COMMAND   PID USER
bash     1234 root
vim      2345 user
```

---

### 方法二：`fuser`

```bash
fuser -vm /mnt/data
```

输出示例：

```
                     USER      PID ACCESS COMMAND
/mnt/data:          root     1234 ..c.. bash
```

如果确定可以结束这些进程：

```bash
fuser -km /mnt/data
```

其中：

* `-k`：结束进程
* `-m`：针对整个挂载点

然后再次：

```bash
umount /mnt/data
```

---

## 六、延迟卸载（Lazy Unmount）

```bash
umount -l /mnt/data
```

特点：

* 立即从目录树中移除挂载点。
* 已经打开该文件的进程可以继续访问，直到关闭文件。
* 新的访问请求无法再进入该挂载点。

适用于：

* 网络文件系统（NFS）
* USB 设备异常
* 无法立即释放资源的场景

需要注意的是，延迟卸载并不会立刻释放底层资源，因此应谨慎使用。

---

## 七、强制卸载

```bash
umount -f /mnt/data
```

主要用于：

* NFS
* 网络存储异常
* 服务器已掉线

对于本地 ext4、xfs 等文件系统，`-f` 通常效果有限，很多情况下仍会提示 busy。

---

## 八、重新挂载为只读

```bash
umount -r /mnt/data
```

如果不能正常卸载：

```
卸载失败
    ↓
自动 remount 为只读
```

适用于：

* 磁盘故障
* 系统维护
* 防止继续写入数据

---

## 九、查看当前挂载情况

在卸载前，可以先查看当前挂载信息：

```bash
mount
```

或者：

```bash
cat /proc/mounts
```

更推荐使用：

```bash
findmnt
```

例如：

```bash
findmnt /mnt/data
```

输出示例：

```
TARGET     SOURCE     FSTYPE OPTIONS
/mnt/data  /dev/sdb1  ext4   rw,relatime
```

---

## 十、实际应用示例

### 卸载 U 盘

```bash
mount /dev/sdb1 /mnt/usb

# 使用完成后
umount /mnt/usb
```

---

### 卸载 NFS

```bash
umount /mnt/nfs
```

若服务器无响应：

```bash
umount -f /mnt/nfs
```

---

### 卸载 Busy 文件系统

```bash
fuser -vm /mnt/data
fuser -km /mnt/data
umount /mnt/data
```

---

## 十一、`umount` 与 `mount` 的关系

| 操作     | 命令                  | 示例                          |
| ------ | ------------------- | --------------------------- |
| 挂载文件系统 | `mount`             | `mount /dev/sdb1 /mnt/data` |
| 卸载文件系统 | `umount`            | `umount /mnt/data`          |
| 查看挂载   | `findmnt` 或 `mount` | `findmnt /mnt/data`         |
| 查看占用进程 | `lsof`、`fuser`      | `fuser -vm /mnt/data`       |

## 十二、注意事项

* **不要直接拔掉 U 盘或移动硬盘**：应先执行 `umount`，确保缓存数据已写回磁盘，避免数据丢失或文件系统损坏。
* **根文件系统不能直接卸载**：系统运行中的根文件系统（`/`）无法直接 `umount`，通常需要进入救援模式、单用户模式或使用 Live CD/USB 后再操作。
* **当前工作目录不能位于待卸载的挂载点**：如果当前 Shell 位于该目录，会导致提示 `target is busy`。
* **生产环境谨慎使用 `-l` 和 `-f`**：`-l`（延迟卸载）和 `-f`（强制卸载）虽然可以解决部分卸载问题，但可能隐藏资源占用或网络故障，应优先排查原因后再使用。