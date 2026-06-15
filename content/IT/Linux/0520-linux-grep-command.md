---
title: Linux grep 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnIli
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/52'
---

# Linux grep 命令

<img src="images/Linux.svg" width="300">

`grep` 是 Linux 中用来**查找文本内容**的命令。它常用于在文件中搜索某个关键词或匹配某种规则。

## 基本用法

```bash
grep "关键词" 文件名
```

例如：

```bash
grep "error" app.log
```

表示在 `app.log` 文件中查找包含 `error` 的行。

## 常用参数

```bash
grep -i "error" app.log
```

忽略大小写。

```bash
grep -n "error" app.log
```

显示匹配行的行号。

```bash
grep -r "error" /var/log/
```

递归搜索目录下所有文件。

```bash
grep -v "debug" app.log
```

显示不包含 `debug` 的行。

```bash
grep -c "error" app.log
```

统计匹配行数。

```bash
grep -E "error|fail|warning" app.log
```

使用扩展正则，匹配多个关键词。

## 常见组合

查看日志中的错误：

```bash
grep -i "error" /var/log/syslog
```

在当前目录所有文件中查找关键词：

```bash
grep -r "数据库连接失败" .
```

查找但排除某些内容：

```bash
grep "ERROR" app.log | grep -v "timeout"
```

## 简单理解

`grep` 就像“文本筛子”：
把文件内容倒进去，只留下符合条件的行。
