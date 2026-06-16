---
title: Linux chown 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKJr
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/72'
---

# Linux chown 命令

<img src="images/Linux.svg" width="300">

`chown` 是 Linux 中用于**修改文件或目录所属用户和所属用户组**的命令。

## 基本语法

```bash
chown [选项] 用户[:用户组] 文件或目录
```

## 常见用法

修改文件所属用户：

```bash
chown user1 file.txt
```

修改文件所属用户和用户组：

```bash
chown user1:group1 file.txt
```

只修改所属用户组：

```bash
chown :group1 file.txt
```

递归修改目录及其内部文件：

```bash
chown -R user1:group1 /data/app
```

## 查看当前所属关系

```bash
ls -l file.txt
```

示例输出：

```bash
-rw-r--r-- 1 root root 1234 Jun 16 file.txt
```

其中第一个 `root` 是所属用户，第二个 `root` 是所属用户组。

## 常用选项

```bash
-R
```

递归处理目录及子目录。

```bash
-v
```

显示处理过程。

```bash
-c
```

只在发生改变时显示信息。

## 实用示例

把 `/var/www/html` 目录交给 `www-data` 用户和用户组：

```bash
sudo chown -R www-data:www-data /var/www/html
```

把某个脚本改为当前用户所有：

```bash
sudo chown $USER:$USER script.sh
```

## 注意事项

`chown` 通常需要 `sudo` 权限。
不要随意对系统目录执行递归修改，例如：

```bash
sudo chown -R user:user /
```

这会破坏系统文件权限，可能导致系统无法正常运行。
