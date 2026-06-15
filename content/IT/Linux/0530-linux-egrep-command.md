---
title: Linux egrep 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnImf
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/53'
---

# Linux egrep 命令

<img src="images/Linux.svg" width="300">

`egrep` 是 Linux 中用来**按扩展正则表达式搜索文本**的命令。

现在更推荐写法是：

```bash
grep -E "正则表达式" 文件名
```

`egrep` 基本等同于：

```bash
grep -E
```

## 基本用法

```bash
egrep "pattern" file.txt
```

例如，查找包含 `error` 的行：

```bash
egrep "error" app.log
```

## 常用示例

查找 `error` 或 `failed`：

```bash
egrep "error|failed" app.log
```

查找以 `root` 开头的行：

```bash
egrep "^root" /etc/passwd
```

查找以 `.conf` 结尾的行：

```bash
egrep "\.conf$" list.txt
```

查找包含数字的行：

```bash
egrep "[0-9]+" file.txt
```

忽略大小写：

```bash
egrep -i "error|warning" app.log
```

显示行号：

```bash
egrep -n "error" app.log
```

递归查找目录：

```bash
egrep -r "password|secret" /etc/
```

只显示匹配内容：

```bash
egrep -o "[0-9]{3,}" file.txt
```

## 常用参数

| 参数   | 作用        |
| ---- | --------- |
| `-i` | 忽略大小写     |
| `-n` | 显示行号      |
| `-v` | 显示不匹配的行   |
| `-r` | 递归搜索目录    |
| `-o` | 只显示匹配内容   |
| `-c` | 统计匹配行数    |
| `-l` | 只显示匹配的文件名 |

## 注意

现在很多系统会提示 `egrep` 已过时，建议使用：

```bash
grep -E "pattern" file.txt
```

日常使用中，把 `egrep` 理解为“支持更强正则表达式的 grep”即可。
