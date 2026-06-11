---
title: Linux tac 命令
section: IT
category: Linux
---

# Linux tac 命令

<img src="images/Linux.svg" width="300">

`tac` 是 Linux 中一个比较实用但不太常用的命令，它的作用可以简单理解为：

> **按行倒序输出文件内容**（`cat` 的反向版本）。

命令名称来源于：

* `cat`：正序显示文件内容
* `tac`：`cat` 反过来拼写

---

## 基本语法

```bash
tac 文件名
```

例如有文件 `test.txt`：

```text
line1
line2
line3
line4
```

执行：

```bash
tac test.txt
```

输出：

```text
line4
line3
line2
line1
```

---

## 与 cat 的区别

### cat

```bash
cat test.txt
```

输出：

```text
line1
line2
line3
line4
```

### tac

```bash
tac test.txt
```

输出：

```text
line4
line3
line2
line1
```

---

## 常见应用场景

### 1. 查看日志最后的内容

日志文件通常不断追加：

```bash
tac app.log | less
```

这样打开时先看到最新日志，而不是最早日志。

相比：

```bash
tail -100 app.log
```

`tac` 可以查看整个文件的倒序内容。

---

### 2. 查找最后一次出现的记录

例如日志：

```text
user1
user2
user3
user1
```

查找最后一次出现的 `user1`：

```bash
tac app.log | grep -m 1 "user1"
```

输出：

```text
user1
```

由于文件已经倒序，因此找到的就是最后一次出现的记录。

---

### 3. 倒序处理数据

例如：

```bash
cat numbers.txt
```

内容：

```text
1
2
3
4
5
```

执行：

```bash
tac numbers.txt | awk '{print $1*10}'
```

输出：

```text
50
40
30
20
10
```

---

## 使用分隔符

默认情况下，`tac` 以换行符作为记录分隔符。

可以通过 `-s` 指定分隔符。

例如文件：

```text
aaa###bbb###ccc###ddd
```

执行：

```bash
tac -s '###' file.txt
```

输出：

```text
ddd###ccc###bbb###aaa
```

---

## 分隔符放在前面

使用 `-b` 参数：

```bash
tac -b -s '###' file.txt
```

可以改变分隔符与内容的位置关系。

这个选项主要用于处理特殊格式文件。

---

## 处理标准输入

`tac` 不仅可以处理文件，也可以处理管道输入：

```bash
echo -e "A\nB\nC" | tac
```

输出：

```text
C
B
A
```

---

## 与 tail 的区别

| 命令     | 功能          |
| ------ | ----------- |
| `tail` | 查看文件末尾若干行   |
| `tac`  | 将整个文件按行倒序输出 |
| `cat`  | 正序输出整个文件    |

例如：

```bash
tail -3 app.log
```

只显示最后 3 行。

```bash
tac app.log
```

显示整个文件，但顺序反转。

---

## 性能说明

对于超大日志文件（几十 GB 以上）：

```bash
tac huge.log
```

通常比：

```bash
cat huge.log | sort -r
```

效率高得多，因为 `tac` 不需要排序，只是从文件尾部向前读取并输出。

---

## 实际运维示例

查看最新错误日志：

```bash
tac /var/log/messages | grep -m 20 ERROR
```

查看最近登录记录：

```bash
tac /var/log/secure | grep sshd
```

查看最后一次系统重启信息：

```bash
tac /var/log/messages | grep -m 1 reboot
```

这些场景中，`tac` 经常与 `grep`、`less`、`awk`、`sed` 等命令组合使用，能够快速从日志尾部开始定位问题。