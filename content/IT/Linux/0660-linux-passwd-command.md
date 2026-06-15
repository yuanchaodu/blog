---
title: Linux passwd 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnI1h
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/66'
---

# Linux passwd 命令

<img src="images/Linux.svg" width="300">

`passwd` 是 Linux 中用于**修改用户密码**的命令，也是系统管理员管理用户密码最常用的工具之一。

## 基本语法

```bash
passwd [选项] [用户名]
```

如果不指定用户名，则修改当前登录用户的密码。

---

## 常见用法

### 1. 修改当前用户密码

```bash
passwd
```

执行后会提示：

```text
Changing password for user nick.
Current password:
New password:
Retype new password:
```

需要先输入旧密码，再输入两次新密码。

---

### 2. root 修改其他用户密码

例如修改用户 `test` 的密码：

```bash
passwd test
```

系统会直接要求输入新密码：

```text
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

不需要知道原密码。

---

### 3. 锁定用户账号

禁止用户使用密码登录：

```bash
passwd -l test
```

查看 `/etc/shadow` 会发现密码字段前增加了 `!`。

解锁：

```bash
passwd -u test
```

---

### 4. 删除密码

使用户登录时不需要密码（一般不建议）：

```bash
passwd -d test
```

---

### 5. 强制用户下次登录修改密码

```bash
passwd -e test
```

或者：

```bash
chage -d 0 test
```

用户下次登录时会看到：

```text
You are required to change your password immediately
```

---

### 6. 查看密码状态

```bash
passwd -S test
```

示例输出：

```text
test PS 2025-06-15 0 99999 7 -1
```

字段说明：

| 字段    | 含义                  |
| ----- | ------------------- |
| test  | 用户名                 |
| PS    | Password Set（已设置密码） |
| 日期    | 最后修改密码日期            |
| 0     | 最短修改间隔天数            |
| 99999 | 密码最长有效期             |
| 7     | 提前 7 天提醒            |
| -1    | 密码失效宽限期             |

状态值：

| 状态 | 含义    |
| -- | ----- |
| PS | 已设置密码 |
| NP | 无密码   |
| LK | 已锁定   |

---

## 密码有效期管理

### 设置密码90天过期

```bash
passwd -x 90 test
```

### 设置最少7天后才能修改

```bash
passwd -n 7 test
```

### 提前15天提醒

```bash
passwd -w 15 test
```

### 设置密码过期后30天禁用账户

```bash
passwd -i 30 test
```

查看效果：

```bash
chage -l test
```

---

## 实际运维常用命令

### 新建用户并设置密码

```bash
useradd appuser
passwd appuser
```

---

### 批量修改密码（脚本方式）

```bash
echo "test:Password@123" | chpasswd
```

多个用户：

```bash
cat users.txt | chpasswd
```

文件格式：

```text
user1:Password@123
user2:Password@123
user3:Password@123
```

---

## 密码存储位置

用户密码信息并不保存在：

```text
/etc/passwd
```

而是保存在：

```text
/etc/shadow
```

查看：

```bash
sudo cat /etc/shadow
```

示例：

```text
test:$6$abc123...:20254:0:99999:7:::
```

其中：

* `$6$`：SHA-512 加密算法
* 后面为加密后的密码哈希值
* 系统无法直接还原出原密码

---

## 常见故障

### passwd: Authentication token manipulation error

通常原因：

1. `/etc/passwd` 或 `/etc/shadow` 权限异常
2. 文件系统被挂载为只读
3. 磁盘空间已满

检查：

```bash
ls -l /etc/passwd /etc/shadow
mount | grep " / "
df -h
```

---

### root 忘记密码

可通过 GRUB 单用户模式进入系统后执行：

```bash
passwd root
```

重新设置 root 密码。

---

## 常用参数速查

| 命令                  | 作用         |
| ------------------- | ---------- |
| `passwd`            | 修改当前用户密码   |
| `passwd user`       | 修改指定用户密码   |
| `passwd -l user`    | 锁定账号       |
| `passwd -u user`    | 解锁账号       |
| `passwd -d user`    | 删除密码       |
| `passwd -e user`    | 强制下次登录修改密码 |
| `passwd -S user`    | 查看密码状态     |
| `passwd -x 90 user` | 设置90天过期    |
| `passwd -n 7 user`  | 最少7天后才能修改  |
| `passwd -w 15 user` | 提前15天提醒    |
| `passwd -i 30 user` | 过期30天后禁用   |

对于日常 Linux 运维来说，`passwd` 主要用于三类工作：**修改密码、锁定/解锁账户、管理密码有效期**。通常会配合 `useradd`、`usermod`、`chage` 和 `/etc/shadow` 一起使用。
