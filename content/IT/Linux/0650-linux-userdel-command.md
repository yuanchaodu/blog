---
title: Linux userdel 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnI05
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/65'
---

# Linux userdel 命令

<img src="images/Linux.svg" width="300">

`userdel` 是 Linux 中用于**删除用户账号**的命令。

## 基本语法

```bash
userdel [选项] 用户名
```

## 常用用法

删除用户，但保留用户家目录：

```bash
sudo userdel zhangsan
```

删除用户，并同时删除家目录和邮件目录：

```bash
sudo userdel -r zhangsan
```

强制删除用户，即使该用户正在登录或有进程：

```bash
sudo userdel -f zhangsan
```

一般不建议直接用 `-f`，可能造成系统中残留文件或进程异常。

## 常用选项

| 选项   | 作用                |
| ---- | ----------------- |
| `-r` | 删除用户的家目录和邮件目录     |
| `-f` | 强制删除用户            |
| `-Z` | 删除该用户的 SELinux 映射 |

## 注意事项

删除用户前，建议先确认用户是否存在：

```bash
id zhangsan
```

查看该用户是否正在登录：

```bash
who
```

查看该用户是否有正在运行的进程：

```bash
ps -u zhangsan
```

如有进程，建议先停止相关进程，再删除用户。

## 实用示例

完整删除用户：

```bash
sudo userdel -r zhangsan
```

删除后检查：

```bash
id zhangsan
```

如果提示 `no such user`，说明用户已删除。
