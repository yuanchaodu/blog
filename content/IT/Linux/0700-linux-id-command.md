---
title: Linux id 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKAm
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/70'
---

# Linux id 命令

<img src="images/Linux.svg" width="300">

`id` 是 Linux 中用于查看用户身份信息的常用命令，可以显示当前用户或指定用户的 UID（用户ID）、GID（组ID）以及所属组信息。

## 基本用法

```bash
id
```

示例输出：

```bash
uid=1000(nick) gid=1000(nick) groups=1000(nick),27(sudo),999(docker)
```

含义：

* `uid=1000(nick)`：用户ID为1000，用户名为nick
* `gid=1000(nick)`：主组ID为1000，组名为nick
* `groups=...`：用户所属的所有组

---

## 查看指定用户

```bash
id root
```

示例：

```bash
uid=0(root) gid=0(root) groups=0(root)
```

---

## 常用参数

### 1. 只显示用户ID

```bash
id -u
```

输出：

```bash
1000
```

查看其他用户：

```bash
id -u root
```

输出：

```bash
0
```

---

### 2. 只显示组ID

```bash
id -g
```

输出：

```bash
1000
```

---

### 3. 显示所有组ID

```bash
id -G
```

输出：

```bash
1000 27 999
```

---

### 4. 显示用户名

```bash
id -un
```

输出：

```bash
nick
```

等价于：

```bash
whoami
```

---

### 5. 显示主组名称

```bash
id -gn
```

输出：

```bash
nick
```

---

### 6. 显示所属组名称

```bash
id -Gn
```

输出：

```bash
nick sudo docker
```

---

## 在脚本中的典型应用

### 判断是否为 root 用户

```bash
if [ "$(id -u)" -eq 0 ]; then
    echo "当前为 root 用户"
else
    echo "当前不是 root 用户"
fi
```

因为：

* root 用户 UID 固定为 `0`
* 普通用户 UID 通常从 `1000` 开始

---

### 获取当前用户名

```bash
USER_NAME=$(id -un)
echo $USER_NAME
```

---

## 与其他命令的区别

| 命令                  | 作用                |
| ------------------- | ----------------- |
| `id`                | 查看 UID、GID 和组信息   |
| `whoami`            | 仅显示当前用户名          |
| `groups`            | 显示用户所属组           |
| `getent passwd 用户名` | 查看用户详细账户信息        |
| `finger 用户名`        | 查看用户详细资料（部分系统未安装） |

例如：

```bash
whoami
```

输出：

```bash
nick
```

而：

```bash
id
```

输出：

```bash
uid=1000(nick) gid=1000(nick) groups=1000(nick),27(sudo),999(docker)
```

信息更加完整。

---

## 实际运维场景

在 Linux 服务器运维中，`id` 常用于：

* 检查程序运行用户身份
* 排查文件权限问题
* 验证用户是否属于某个组
* 判断脚本是否由 root 执行
* 检查 NFS、Samba、Docker 等服务的用户映射关系

例如检查当前用户是否属于 `docker` 组：

```bash
id -Gn | grep -w docker
```

如果有输出，说明当前用户具备执行 Docker 命令的权限。
