---
title: Linux scp 命令
section: IT
category: Linux
---

# Linux scp 命令

<img src="images/Linux.svg" width="300">

`scp`（**Secure Copy**）是 Linux 中基于 SSH 协议进行文件传输的命令，可以安全地在本地和远程主机之间复制文件或目录。只要能够通过 SSH 登录服务器，通常就可以使用 `scp`。

## 基本语法

```bash
scp [选项] 源文件 目标文件
```

常见格式：

```bash
scp [选项] source destination
```

其中：

* `source`：源文件或目录
* `destination`：目标文件或目录

远程主机的格式为：

```text
用户名@主机IP:路径
```

例如：

```text
root@192.168.1.100:/home/root/
```

---

## 常用示例

### 1. 本地复制到远程服务器

将本地文件复制到远程服务器：

```bash
scp test.txt root@192.168.1.100:/home/root/
```

复制完成后：

```
本地
test.txt
    │
    └────► 远程服务器
           /home/root/test.txt
```

---

### 2. 从远程服务器复制到本地

```bash
scp root@192.168.1.100:/home/root/test.txt ./
```

表示：

将远程服务器的 `test.txt` 下载到当前目录。

也可以指定目录：

```bash
scp root@192.168.1.100:/home/root/test.txt /tmp/
```

---

### 3. 复制整个目录

使用 `-r`：

```bash
scp -r project root@192.168.1.100:/data/
```

表示复制整个 `project` 文件夹。

下载目录：

```bash
scp -r root@192.168.1.100:/data/project ./
```

---

### 4. 指定 SSH 端口

如果 SSH 使用的不是默认 22 端口：

```bash
scp -P 2222 test.txt root@192.168.1.100:/tmp/
```

注意：

* **scp 是大写 `-P`**
* **ssh 是小写 `-p`（表示保持时间属性）**

---

### 5. 使用私钥登录

例如：

```bash
scp -i ~/.ssh/id_rsa test.txt root@192.168.1.100:/tmp/
```

---

### 6. 保留文件属性

```bash
scp -p test.txt root@192.168.1.100:/tmp/
```

保留：

* 修改时间
* 访问时间
* 文件权限

---

### 7. 限制传输速度

例如限制为 5MB/s（约 5000 Kbit/s）：

```bash
scp -l 40000 test.iso root@192.168.1.100:/tmp/
```

说明：

`-l` 单位为 **Kbit/s**。

---

### 8. 显示详细调试信息

```bash
scp -v test.txt root@192.168.1.100:/tmp/
```

用于排查：

* SSH 登录问题
* 密钥认证失败
* 网络连接问题

---

## 两台远程服务器之间复制

例如：

```bash
scp root@192.168.1.10:/tmp/a.txt root@192.168.1.20:/tmp/
```

默认情况下，数据通常会通过执行命令的本地主机进行中转。

如果希望源服务器与目标服务器直接传输（前提是两台服务器能够互相 SSH 通信），可以使用：

```bash
scp -3 root@192.168.1.10:/tmp/a.txt root@192.168.1.20:/tmp/
```

> 不同 OpenSSH 版本对远程到远程复制的默认行为有所不同，建议根据实际环境测试，必要时使用 `-3` 明确指定通过本地主机中转。

---

## 常用选项

| 选项   | 说明                                         |
| ---- | ------------------------------------------ |
| `-r` | 递归复制目录                                     |
| `-P` | 指定 SSH 端口                                  |
| `-i` | 指定私钥                                       |
| `-p` | 保留文件属性                                     |
| `-C` | 启用压缩，提高低带宽网络传输效率                           |
| `-l` | 限制带宽（Kbit/s）                               |
| `-v` | 显示详细调试信息                                   |
| `-q` | 安静模式，不显示进度                                 |
| `-o` | 传递 SSH 参数，例如 `-o StrictHostKeyChecking=no` |

---

## 常见错误

### 1. Permission denied

```
Permission denied (publickey,password).
```

原因：

* 用户名错误
* 密码错误
* SSH 公钥未配置
* 没有目标目录权限

---

### 2. No such file or directory

```
scp: /tmp/test.txt: No such file or directory
```

说明目标路径不存在。

---

### 3. Connection refused

```
ssh: connect to host 192.168.1.100 port 22: Connection refused
```

说明：

* SSH 服务未启动
* 端口错误
* 防火墙阻止连接

---

## `scp` 与 `rsync` 的比较

| 功能       | `scp`         | `rsync`       |
| -------- | ------------- | ------------- |
| 安全传输     | ✓（SSH）        | ✓（SSH）        |
| 增量同步     | ✗             | ✓             |
| 断点续传     | ✗（新版本已移除相关支持） | ✓             |
| 同步目录     | 一般            | 非常适合          |
| 速度       | 普通            | 大量文件或重复同步时更快  |
| 删除目标多余文件 | ✗             | ✓（`--delete`） |

如果只是偶尔复制文件，`scp` 简单方便；如果需要频繁同步目录、大量文件或希望支持增量同步、断点续传等功能，通常推荐使用 `rsync`。