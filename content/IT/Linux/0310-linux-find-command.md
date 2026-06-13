---
title: Linux find 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDSb
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/31'
---

# Linux find 命令

<img src="images/Linux.svg" width="300">

`find` 是 Linux 中用来**查找文件和目录**的命令，功能很强。

## 基本格式

```bash
find 查找路径 查找条件 执行动作
```

例如：

```bash
find /home -name "*.txt"
```

意思是：在 `/home` 目录下查找所有 `.txt` 文件。

## 常用示例

按名称查找：

```bash
find /var/log -name "*.log"
```

限定深度
```bash
find . -maxdepth 1 -name "*.csv"
```

忽略大小写：

```bash
find . -iname "readme*"
```

查找目录：

```bash
find /etc -type d -name "nginx"
```

查找普通文件：

```bash
find . -type f
```

按大小查找：

```bash
find . -size +100M
```

表示查找大于 100MB 的文件。

按修改时间查找：

```bash
find . -mtime -7
```

表示查找 7 天内修改过的文件。

查找空文件：

```bash
find . -type f -empty
```

查找后删除：

```bash
find . -name "*.tmp" -delete
```

使用前建议先不加 `-delete`，确认结果没问题再删除。

## 配合执行命令

查找并显示详细信息：

```bash
find . -name "*.sh" -exec ls -l {} \;
```

查找并修改权限：

```bash
find . -type f -name "*.sh" -exec chmod +x {} \;
```

其中 `{}` 代表找到的文件。

## 常用条件速查

| 条件        | 作用          |
| --------- | ----------- |
| `-name`   | 按名称查找       |
| `-iname`  | 按名称查找，忽略大小写 |
| `-type f` | 查找文件        |
| `-type d` | 查找目录        |
| `-size`   | 按大小查找       |
| `-mtime`  | 按修改时间查找     |
| `-user`   | 按所属用户查找     |
| `-perm`   | 按权限查找       |
| `-empty`  | 查找空文件或空目录   |
| `-delete` | 删除查找到的内容    |
| `-exec`   | 对查找到的内容执行命令 |

最常用的是：

```bash
find . -name "文件名"
```

`.` 表示从当前目录开始查找。
