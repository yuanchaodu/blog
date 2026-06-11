---
title: Linux more 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnD5p
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/43'
---

# Linux more 命令

<img src="images/Linux.svg" width="300">

`more` 是 Linux 中最早、最常用的分页查看命令之一，用于按页显示文本内容，避免文件内容一次性全部输出到终端。

## 基本语法

```bash
more [选项] 文件名
```

例如：

```bash
more /etc/passwd
```

会按页显示 `/etc/passwd` 文件内容。

---

## 常见用法

### 查看文件

```bash
more test.log
```

每显示满一屏后暂停，等待用户操作。

### 查看命令输出

结合管道使用：

```bash
ls -l | more
```

或：

```bash
cat large.txt | more
```

---

## 常用操作键

在 `more` 运行过程中，可以使用以下快捷键：

| 按键        | 功能            |
| --------- | ------------- |
| Space（空格） | 向下翻一页         |
| Enter     | 向下滚动一行        |
| b         | 返回上一页（部分系统支持） |
| /字符串      | 搜索指定内容        |
| n         | 查找下一个匹配项      |
| q         | 退出 more       |
| h         | 显示帮助          |

例如：

```text
--More--
```

出现后：

* 按空格 → 下一页
* 按回车 → 下一行
* 按 q → 退出

---

## 常用选项

### 从指定行开始显示

```bash
more +100 test.log
```

从第100行开始查看。

### 从匹配内容开始显示

```bash
more +/ERROR test.log
```

从第一次出现 `ERROR` 的位置开始显示。

### 显示行号

```bash
more -N test.log
```

部分 Linux 发行版支持显示行号。

---

## 实际应用示例

### 查看大型日志文件

```bash
more /var/log/messages
```

### 查看配置文件

```bash
more /etc/ssh/sshd_config
```

### 查看长命令输出

```bash
ps -ef | more
```

```bash
netstat -an | more
```

---

## more 与 less 的区别

| 功能    | more | less |
| ----- | ---- | ---- |
| 向下翻页  | √    | √    |
| 向上翻页  | 有限支持 | √    |
| 搜索    | √    | √    |
| 性能    | 一般   | 更好   |
| 功能丰富度 | 较少   | 丰富   |
| 推荐程度  | 一般   | 推荐   |

现代 Linux 系统中，更推荐使用：

```bash
less 文件名
```

因为 `less` 功能更强，支持：

* 前后翻页
* 更方便搜索
* 跳转到指定行
* 彩色显示（配合其他工具）
* 更适合查看超大日志文件

例如：

```bash
less /var/log/messages
```

很多 Linux 管理员日常已经基本用 `less` 替代 `more`。

---

## 一句话总结

`more` 可以理解为 Linux 中的“分页阅读器”，适合查看较长的文本文件：

```bash
more 文件名
```

常用操作：

* 空格：下一页
* 回车：下一行
* `/关键字`：搜索
* `q`：退出

如果经常查看日志或配置文件，建议优先掌握 `less` 命令。
