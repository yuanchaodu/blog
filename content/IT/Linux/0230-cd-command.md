---
title: Linux cd 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnCAy
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/23'
---

# Linux cd 命令

<img src="images/Linux.svg" width="300">

`cd`（Change Directory）是 Linux 中最常用的命令之一，用于**切换当前工作目录**。

## 基本语法

```bash
cd [目录路径]
```

例如：

```bash
cd /home/nick
```

表示切换到 `/home/nick` 目录。

---

## 常见用法

### 1. 切换到指定目录

```bash
cd /etc
```

进入 `/etc` 目录。

---

### 2. 切换到当前用户的家目录

```bash
cd
```

或者：

```bash
cd ~
```

例如：

```bash
cd ~
```

进入当前用户的家目录：

```text
/home/username
```

---

### 3. 返回上一级目录

```bash
cd ..
```

例如：

当前目录：

```text
/home/nick/project
```

执行：

```bash
cd ..
```

变为：

```text
/home/nick
```

---

### 4. 返回上两级目录

```bash
cd ../..
```

例如：

```text
/home/nick/project/test
```

执行后变为：

```text
/home/nick
```

---

### 5. 切换到根目录

```bash
cd /
```

根目录是 Linux 文件系统的最高层。

---

### 6. 返回刚才所在目录

```bash
cd -
```

例如：

```bash
cd /etc
cd /var
cd -
```

会返回：

```text
/etc
```

再次执行：

```bash
cd -
```

又会回到：

```text
/var
```

类似于两个目录之间来回切换。

---

### 7. 使用相对路径

当前目录：

```text
/home/nick
```

执行：

```bash
cd project
```

进入：

```text
/home/nick/project
```

这里的 `project` 就是相对路径。

---

### 8. 使用绝对路径

```bash
cd /home/nick/project
```

从任何位置执行都会进入该目录。

---

## 常用目录符号

| 符号   | 含义      |
| ---- | ------- |
| `.`  | 当前目录    |
| `..` | 上一级目录   |
| `~`  | 当前用户家目录 |
| `/`  | 根目录     |
| `-`  | 上一次所在目录 |

---

## 配合 pwd 查看当前位置

切换目录后，经常会配合 `pwd` 使用：

```bash
pwd
```

例如：

```bash
cd /etc
pwd
```

输出：

```text
/etc
```

---

## 实际工作中的常见操作

进入日志目录：

```bash
cd /var/log
```

进入应用目录：

```bash
cd /opt/app
```

回到家目录：

```bash
cd ~
```

返回上一次目录：

```bash
cd -
```

查看当前目录：

```bash
pwd
```

---

## 小技巧

### 自动补全

输入部分目录名后按 **Tab** 键：

```bash
cd /var/lo[TAB]
```

自动补全为：

```bash
cd /var/log
```

如果有多个匹配项，可以连续按两次 `Tab` 查看候选目录。

### 包含空格的目录

目录名有空格时：

```bash
cd "My Documents"
```

或者：

```bash
cd My\ Documents
```

---

## 一张图理解

```text
/
├── home
│   └── nick
│       └── project
├── etc
└── var
```

如果当前在：

```text
/home/nick/project
```

命令效果如下：

```bash
cd ..      -> /home/nick
cd ../..   -> /home
cd /etc    -> /etc
cd ~       -> /home/nick
cd /       -> /
cd -       -> 返回上一个目录
```

掌握 `cd`、`ls`、`pwd` 这三个命令后，基本就能完成 Linux 文件系统中的大部分导航操作。
