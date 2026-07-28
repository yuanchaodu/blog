---
title: Linux shell 管道
section: IT
category: Linux
---

# Linux shell 管道

<img src="images/Linux.svg" width="300">

## Linux Shell 管道

Linux Shell 管道是一种把多个命令连接起来的机制。它使用竖线符号：

```bash
|
```

基本格式：

```bash
命令1 | 命令2 | 命令3
```

它表示：**将前一个命令的输出，作为后一个命令的输入。**

可以把管道理解成一条流水线。前一道工序处理完数据后，立即交给下一道工序继续处理。

---

## 一个简单例子

查看当前目录中的文件：

```bash
ls
```

假设只想查找名称中包含 `log` 的文件，可以写：

```bash
ls | grep log
```

执行过程是：

1. `ls` 输出当前目录的文件列表。
2. 管道 `|` 把文件列表交给 `grep`。
3. `grep log` 只保留包含 `log` 的行。

---

## 常见用法

### 1. 查找进程

```bash
ps aux | grep nginx
```

含义：

* `ps aux`：列出系统进程。
* `grep nginx`：筛选包含 `nginx` 的进程。

为了避免显示 `grep nginx` 自己，可以使用：

```bash
ps aux | grep '[n]ginx'
```

也可以使用更直接的命令：

```bash
pgrep -a nginx
```

---

### 2. 统计文件数量

```bash
ls -1 | wc -l
```

含义：

* `ls -1`：每行显示一个文件名。
* `wc -l`：统计行数。

不过，这种方式不能可靠处理特殊文件名。更稳妥的方式是：

```bash
find . -maxdepth 1 -type f | wc -l
```

---

### 3. 查看日志中的错误

```bash
cat app.log | grep ERROR
```

虽然这样可以执行，但 `cat` 是多余的。更简洁的写法是：

```bash
grep ERROR app.log
```

进一步统计错误数量：

```bash
grep ERROR app.log | wc -l
```

---

### 4. 对结果排序并去重

```bash
cat names.txt | sort | uniq
```

更简洁地写：

```bash
sort names.txt | uniq
```

也可以直接使用：

```bash
sort -u names.txt
```

如果需要统计每个名称出现的次数：

```bash
sort names.txt | uniq -c
```

---

### 5. 提取指定字段

假设文件内容如下：

```text
zhangsan 25 IT
lisi 30 HR
wangwu 28 IT
```

提取第一列姓名：

```bash
awk '{print $1}' users.txt
```

只查看 IT 部门人员：

```bash
awk '$3 == "IT" {print $1}' users.txt
```

也可以使用管道：

```bash
grep ' IT$' users.txt | awk '{print $1}'
```

---

## 管道连接的是什么

管道主要连接两个程序的：

* 标准输出：`stdout`
* 标准输入：`stdin`

默认情况下，管道只传递标准输出，不传递错误输出。

例如：

```bash
command1 | command2
```

等价于：

```text
command1 的标准输出 → command2 的标准输入
```

---

## 错误输出如何进入管道

普通管道不会传递标准错误：

```bash
command 2>/dev/null | grep keyword
```

这里：

* `2>/dev/null`：丢弃错误信息。
* 正常输出继续进入管道。

如果希望正常输出和错误输出都进入管道，可以写：

```bash
command 2>&1 | grep keyword
```

在 Bash 中还可以写：

```bash
command |& grep keyword
```

例如：

```bash
find / -name "*.conf" 2>&1 | grep "Permission denied"
```

这条命令会筛选查找过程中产生的权限错误。

---

## 管道与重定向的区别

管道是把输出交给另一个命令：

```bash
ls | grep txt
```

重定向是把输出写入文件：

```bash
ls > files.txt
```

二者可以组合使用：

```bash
ps aux | grep nginx > nginx_processes.txt
```

含义是：

1. 获取进程列表。
2. 筛选 nginx。
3. 把结果写入文件。

如果希望追加到文件，而不是覆盖文件：

```bash
ps aux | grep nginx >> nginx_processes.txt
```

---

## 使用 `tee` 同时显示和保存

普通重定向后，结果通常不会显示在终端：

```bash
ls | grep log > result.txt
```

使用 `tee` 可以同时显示并保存：

```bash
ls | grep log | tee result.txt
```

追加写入文件：

```bash
ls | grep log | tee -a result.txt
```

例如，一边查看日志，一边保存：

```bash
tail -f app.log | tee monitor.log
```

---

## 管道的退出状态

默认情况下，一个管道命令的退出状态通常取决于最后一个命令。

例如：

```bash
false | true
echo $?
```

结果通常是：

```text
0
```

虽然前面的 `false` 失败了，但最后的 `true` 成功，因此整个管道看起来成功。

在脚本中，建议启用：

```bash
set -o pipefail
```

启用后，只要管道中有命令失败，整个管道就会返回失败状态。

常见脚本设置：

```bash
set -euo pipefail
```

其中：

* `-e`：命令失败时退出脚本。
* `-u`：使用未定义变量时报错。
* `pipefail`：管道中的失败不会被忽略。

---

## 一个综合例子

统计访问日志中出现次数最多的前 10 个 IP：

```bash
awk '{print $1}' access.log |
sort |
uniq -c |
sort -nr |
head -10
```

执行过程：

1. `awk '{print $1}'`：提取第一列 IP 地址。
2. `sort`：把相同 IP 排到一起。
3. `uniq -c`：统计每个 IP 的出现次数。
4. `sort -nr`：按次数从大到小排序。
5. `head -10`：显示前 10 个结果。

写成一行：

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

这就是管道的核心价值：**让每个命令只负责一件小事，再把这些小工具组合成完整的数据处理流程。**