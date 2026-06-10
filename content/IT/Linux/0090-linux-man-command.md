---
title: Linux man 命令详解
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnBy9
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/9'
---

# Linux man 命令详解

<img src="images/Linux.svg" width="300">

`man` 是 Linux 中查看“帮助手册”的命令，相当于系统自带说明书。

## 一、基本用法

```bash
man 命令名
```

例如：

```bash
man ls
```

查看 `ls` 命令的详细说明。

退出手册：

```bash
q
```

## 二、常用操作键

进入 `man` 页面后：

| 按键      | 作用     |
| ------- | ------ |
| `空格`    | 向下翻一页  |
| `b`     | 向上翻一页  |
| `Enter` | 向下滚动一行 |
| `/关键词`  | 向下搜索   |
| `n`     | 查找下一个  |
| `N`     | 查找上一个  |
| `q`     | 退出     |

## 三、常用参数

### 1. 查看命令手册

```bash
man ls
```

### 2. 搜索相关手册

```bash
man -k keyword
```

等同于：

```bash
apropos keyword
```

例如：

```bash
man -k copy
```

会搜索和 `copy` 相关的手册。

### 3. 查看指定章节

```bash
man 章节号 名称
```

例如：

```bash
man 5 passwd
```

查看 `/etc/passwd` 文件格式说明。

而：

```bash
man passwd
```

通常查看的是 `passwd` 命令说明。

### 4. 查看手册所在位置

```bash
man -w ls
```

### 5. 查看所有匹配手册

```bash
man -a printf
```

可能会依次显示 `printf` 命令、C 函数等不同手册。

## 四、man 手册章节

| 章节 | 内容     |
| -- | ------ |
| 1  | 用户命令   |
| 2  | 系统调用   |
| 3  | C 库函数  |
| 4  | 设备文件   |
| 5  | 文件格式   |
| 6  | 游戏     |
| 7  | 杂项说明   |
| 8  | 系统管理命令 |
| 9  | 内核相关内容 |

常见例子：

```bash
man 1 printf
man 3 printf
```

一个是 shell 命令，一个是 C 语言函数。

## 五、man 页面结构

常见栏目包括：

| 栏目          | 含义      |
| ----------- | ------- |
| NAME        | 名称和简短说明 |
| SYNOPSIS    | 使用格式    |
| DESCRIPTION | 详细说明    |
| OPTIONS     | 参数选项    |
| EXAMPLES    | 示例      |
| FILES       | 相关文件    |
| SEE ALSO    | 相关命令    |
| AUTHOR      | 作者      |
| BUGS        | 已知问题    |

## 六、实用示例

查看 `grep` 用法：

```bash
man grep
```

在页面中搜索忽略大小写参数：

```bash
/-i
```

查看 `crontab` 文件格式：

```bash
man 5 crontab
```

查看 `systemctl` 说明：

```bash
man systemctl
```

搜索和网络相关的手册：

```bash
man -k network
```

## 七、和其他帮助命令的区别

| 命令          | 作用         |
| ----------- | ---------- |
| `man`       | 最完整的系统手册   |
| `命令 --help` | 快速查看参数     |
| `info`      | GNU 风格详细文档 |
| `whatis`    | 查看一句话说明    |
| `apropos`   | 按关键词搜索手册   |

## 八、常用组合建议

日常排查命令参数时：

```bash
命令 --help
man 命令
```

不确定命令名时：

```bash
man -k 关键词
apropos 关键词
```

要区分命令、配置文件、函数时：

```bash
man 章节号 名称
```

`man` 最适合查“准确用法”和“参数含义”，是 Linux 运维和开发中非常基础也非常重要的工具。
