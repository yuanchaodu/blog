---
title: Linux usermod 命令
section: IT
category: Linux
---

# Linux usermod 命令

<img src="images/Linux.svg" width="300">

`usermod` 是 Linux 中用于**修改已有用户账号属性**的命令。

常见用法：

```bash
usermod [选项] 用户名
```

常用参数：

| 参数    | 作用              | 示例                                     |
| ----- | --------------- | -------------------------------------- |
| `-l`  | 修改用户名           | `usermod -l newname oldname`           |
| `-u`  | 修改 UID          | `usermod -u 1002 zhangsan`             |
| `-g`  | 修改主组            | `usermod -g group1 zhangsan`           |
| `-G`  | 修改附加组           | `usermod -G wheel,docker zhangsan`     |
| `-aG` | 追加附加组，不覆盖原有组    | `usermod -aG docker zhangsan`          |
| `-d`  | 修改家目录路径         | `usermod -d /home/newhome zhangsan`    |
| `-m`  | 配合 `-d`，迁移家目录内容 | `usermod -d /home/newhome -m zhangsan` |
| `-s`  | 修改登录 Shell      | `usermod -s /bin/bash zhangsan`        |
| `-L`  | 锁定用户账号          | `usermod -L zhangsan`                  |
| `-U`  | 解锁用户账号          | `usermod -U zhangsan`                  |
| `-e`  | 设置账号过期日期        | `usermod -e 2026-12-31 zhangsan`       |

最常用例子：

```bash
usermod -aG docker zhangsan
```

表示把用户 `zhangsan` 加入 `docker` 组，且不影响原来的附加组。

注意：
使用 `-G` 时，如果不加 `-a`，会**覆盖用户原来的附加组**。所以实际工作中更常用：

```bash
usermod -aG 组名 用户名
```

查看用户所属组：

```bash
groups zhangsan
```

查看用户信息：

```bash
id zhangsan
```