---
title: Linux groupmod 命令
section: IT
category: Linux
---

# Linux groupmod 命令

<img src="images/Linux.svg" width="300">

`groupmod` 是 Linux 中用于**修改已有用户组（Group）信息**的命令，属于用户和组管理工具集。

## 基本语法

```bash
groupmod [选项] 组名
```

## 常用选项

| 选项       | 说明                        |
| -------- | ------------------------- |
| `-n 新组名` | 修改组名称                     |
| `-g GID` | 修改组ID（Group ID）           |
| `-o`     | 允许使用非唯一GID（通常与 `-g` 一起使用） |

---

## 示例

### 1. 修改组名称

将组 `testgroup` 改为 `devgroup`：

```bash
sudo groupmod -n devgroup testgroup
```

修改前：

```bash
grep testgroup /etc/group
```

输出：

```text
testgroup:x:1001:
```

修改后：

```text
devgroup:x:1001:
```

---

### 2. 修改组GID

将组 `devgroup` 的 GID 改为 2000：

```bash
sudo groupmod -g 2000 devgroup
```

查看结果：

```bash
grep devgroup /etc/group
```

输出：

```text
devgroup:x:2000:
```

---

### 3. 使用重复GID

默认情况下 GID 必须唯一。

如果确实需要多个组共享同一个 GID，可以使用：

```bash
sudo groupmod -g 2000 -o devgroup
```

例如：

```bash
sudo groupmod -g 1000 -o group2
```

---

## 查看组信息

### 查看所有组

```bash
cat /etc/group
```

### 查询指定组

```bash
getent group devgroup
```

输出：

```text
devgroup:x:2000:
```

---

## 注意事项

### 1. 修改GID后文件权限可能受影响

Linux 文件记录的是数字 GID，而不是组名。

例如：

```bash
ls -ln /data
```

输出：

```text
-rw-r--r-- 1 1000 1001 file.txt
```

这里的 `1001` 是实际保存的 GID。

如果把组 GID 从 1001 改成 2000：

```bash
groupmod -g 2000 devgroup
```

原有文件仍然保存 GID=1001，导致显示为未知组。

需要同步修改文件所属组：

```bash
find /data -group 1001 -exec chgrp -h devgroup {} \;
```

或者：

```bash
find /data -gid 1001 -exec chgrp devgroup {} \;
```

---

### 2. 不要修改系统保留组

例如：

```text
root
daemon
bin
sys
adm
wheel
systemd-journal
```

修改这些系统组可能导致系统异常。

---

### 3. 用户主组修改需同时处理

如果某用户的主组是被修改的组：

```bash
id username
```

查看：

```text
uid=1001(user1) gid=1001(devgroup)
```

修改组信息后，建议确认用户主组是否正常：

```bash
usermod -g devgroup user1
```

---

## 相关命令

| 命令             | 功能       |
| -------------- | -------- |
| `groupadd`     | 创建用户组    |
| `groupdel`     | 删除用户组    |
| `groupmod`     | 修改用户组    |
| `useradd`      | 创建用户     |
| `usermod`      | 修改用户     |
| `id`           | 查看用户和组信息 |
| `getent group` | 查询组数据库   |
| `gpasswd`      | 管理组成员    |

## 实际运维中的常见场景

例如公司新建应用服务器时，原来有一个组：

```text
app:x:1005:
```

现在统一命名规范，需要改为：

```text
appadmin:x:1005:
```

执行：

```bash
sudo groupmod -n appadmin app
```

验证：

```bash
getent group appadmin
```

即可完成组名变更，而不会影响现有文件权限（因为 GID 未改变）。如果同时修改 GID，则需要额外检查和修复文件所属组。