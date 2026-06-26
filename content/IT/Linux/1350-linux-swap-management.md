---
title: Linux Swap 管理
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnYyd
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/135'
---

# Linux Swap 管理

<img src="images/Linux.svg" width="300">

Linux 中的 **Swap（交换空间）** 是一种虚拟内存机制。当物理内存（RAM）不足时，Linux 会将暂时不用的内存页写入 Swap，从而释放 RAM 给正在运行的程序使用。

不过，现代 Linux 中，Swap 的作用已经不仅仅是“内存不够才使用”，它还参与内存回收、休眠（Hibernate）、OOM（Out Of Memory）保护等机制。

下面从原理、管理、优化三个方面介绍。

---

# 一、查看 Swap 状态

## 1、查看当前 Swap

```bash
swapon --show
```

例如：

```text
NAME      TYPE  SIZE   USED PRIO
/dev/sda2 partition 16G  1.2G   -2
```

或者

```bash
cat /proc/swaps
```

输出：

```text
Filename        Type      Size     Used Priority
/dev/sda2       partition 16777212 204800 -2
```

---

## 2、查看内存和 Swap 使用情况

```bash
free -h
```

例如：

```text
              total   used   free  shared buff/cache available
Mem:           31Gi   12Gi   10Gi    500Mi    9Gi      18Gi
Swap:          16Gi  1.3Gi   15Gi
```

重点：

* **used**：已经使用的 Swap
* **free**：剩余 Swap

---

## 3、查看内存压力

```bash
vmstat 1
```

例如：

```text
procs -----------memory---------- ---swap--
 r  b swpd free buff cache si so
 0  0 1200 200000 ...
```

其中：

* **si**：Swap In（从磁盘读回内存）
* **so**：Swap Out（写入 Swap）

如果：

```
si
so
```

持续增长，说明系统正在频繁交换。

---

# 二、创建 Swap

Linux 支持两种方式：

* Swap 分区（推荐服务器）
* Swap 文件（推荐云服务器）

---

## 方法一：创建 Swap 文件

例如创建 8GB：

```bash
sudo fallocate -l 8G /swapfile
```

如果不支持：

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=8192
```

---

修改权限：

```bash
sudo chmod 600 /swapfile
```

初始化：

```bash
sudo mkswap /swapfile
```

输出：

```
Setting up swapspace version 1
```

启用：

```bash
sudo swapon /swapfile
```

查看：

```bash
swapon --show
```

---

## 开机自动挂载

编辑：

```bash
sudo vi /etc/fstab
```

增加：

```text
/swapfile none swap sw 0 0
```

---

# 三、关闭 Swap

临时关闭：

```bash
sudo swapoff -a
```

关闭指定：

```bash
sudo swapoff /swapfile
```

再次开启：

```bash
sudo swapon -a
```

---

# 四、删除 Swap

关闭：

```bash
swapoff /swapfile
```

删除：

```bash
rm /swapfile
```

删除 `/etc/fstab` 中对应配置。

---

# 五、修改 Swap 大小

Swap 文件不能直接扩容。

例如：

原来：

```
4G
```

改成：

```
16G
```

步骤：

```bash
swapoff /swapfile
rm /swapfile
fallocate -l 16G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

---

# 六、Swap 分区管理

查看：

```bash
lsblk
```

例如：

```
sda
├── sda1 /
├── sda2 [SWAP]
```

启用：

```bash
swapon /dev/sda2
```

关闭：

```bash
swapoff /dev/sda2
```

格式化：

```bash
mkswap /dev/sda2
```

---

# 七、Swappiness（Swap 使用倾向）

Linux 不会等内存完全耗尽才开始使用 Swap，而是根据 **Swappiness** 参数决定回收匿名页（Anonymous Pages）与文件缓存（Page Cache）的策略。

查看：

```bash
cat /proc/sys/vm/swappiness
```

例如：

```
60
```

修改：

```bash
sudo sysctl vm.swappiness=10
```

永久：

编辑：

```bash
/etc/sysctl.conf
```

增加：

```text
vm.swappiness=10
```

生效：

```bash
sudo sysctl -p
```

### 常见建议

| 场景     |   建议值 |
| ------ | ----: |
| 数据库服务器 |  1～10 |
| 应用服务器  | 10～20 |
| 桌面系统   | 20～60 |
| 默认值    |    60 |

> 说明：Swappiness 越低，并不代表“完全不用 Swap”，而是更倾向于保留匿名内存、优先回收文件缓存；只有在内存压力较大时才会更多使用 Swap。

---

# 八、Swap Priority（优先级）

多个 Swap 可以同时存在。

查看：

```bash
swapon --show
```

```
NAME      SIZE  PRIO
swap1      8G     5
swap2      8G    -2
```

优先级：

```bash
swapon -p 10 /swapfile
```

数字越大：

```
10
```

优先使用。

---

# 九、监控 Swap

## top

```bash
top
```

按：

```
Shift + M
```

按内存排序。

---

## htop

```bash
htop
```

直接显示：

```
Mem
Swap
```

---

## sar

安装：

```bash
sudo apt install sysstat
```

查看：

```bash
sar -W 1
```

显示：

```
pswpin/s
pswpout/s
```

---

## iostat

```bash
iostat -x 1
```

观察磁盘是否因 Swap 导致 I/O 繁忙。

---

# 十、排查 Swap 使用过高

即使 `free` 显示还有大量可用内存，Swap 已经使用了一部分也不一定是异常。Linux 可能将长期未访问的内存页放入 Swap，以便把 RAM 留给文件缓存，提高整体性能。

建议按以下步骤排查：

1. 查看整体内存情况：

```bash
free -h
```

2. 查看哪些进程占用内存较多：

```bash
ps aux --sort=-%mem | head
```

3. 查看实时内存变化：

```bash
top
```

或

```bash
htop
```

4. 查看是否存在频繁交换：

```bash
vmstat 1
```

重点关注 `si` 和 `so` 是否持续增长。如果长期接近 0，即使 Swap 有一定占用，也通常无需担心。

---

# 十一、清空已使用的 Swap

有时内存已经充足，但 Swap 中仍保留着历史数据，可以尝试清空：

```bash
sudo swapoff -a
sudo swapon -a
```

执行时需要确保当前有足够的可用 RAM，否则可能导致系统性能下降甚至触发 OOM。

---

# 十二、服务器环境的建议

| 场景          | 建议                                                  |
| ----------- | --------------------------------------------------- |
| 物理服务器       | 建议保留一定 Swap（如 4～16GB），作为内存压力缓冲。                     |
| 云服务器        | 建议使用 Swap 文件，扩容和管理更方便。                              |
| 数据库服务器      | 保留少量 Swap，并将 `vm.swappiness` 调低（如 1～10），避免频繁交换影响性能。 |
| 容器宿主机       | 建议保留适量 Swap，并结合 cgroup 内存限制统一管理。                    |
| 内存充足（≥64GB） | 不建议完全关闭 Swap，可保留 2～8GB，用于异常情况下的缓冲和故障分析（如生成内核转储）。    |

### 实践建议

* **不要仅凭“Swap 已使用”判断系统有问题**，更应关注是否发生持续的 Swap In/Out。
* **监控 `vmstat`、`sar -W` 和磁盘 I/O**，判断是否存在交换抖动（Thrashing）。
* **根据业务特点调整 `vm.swappiness`**，而不是一味设置为 0。
* **定期分析内存占用较高的进程**，优化应用配置或增加物理内存，而不是依赖 Swap 解决内存不足问题。

合理配置和监控 Swap，可以在内存紧张时提升系统稳定性，但如果系统长期大量依赖 Swap，通常意味着需要优化应用或扩充物理内存，而不是单纯扩大 Swap 空间。
