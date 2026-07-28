---
title: Linux 输入输出重定向
section: IT
category: Linux
---

# Linux 输入输出重定向

<img src="images/Linux.svg" width="300">

## Linux 输入输出重定向

Linux 命令运行时，通常会使用三条“通道”：

| 名称          | 编号 | 默认位置 |
| ----------- | -: | ---- |
| 标准输入 stdin  |  0 | 键盘   |
| 标准输出 stdout |  1 | 终端屏幕 |
| 标准错误 stderr |  2 | 终端屏幕 |

所谓“重定向”，就是把这些通道改到文件、其他命令或设备中。

可以把它理解为水管：命令产生的数据像水，重定向符号负责改变水流方向。

---

## 1. 输出重定向 `>`

把命令的正常输出写入文件。

```bash
ls > files.txt
```

执行后，`ls` 的结果不会显示在屏幕上，而是保存到 `files.txt`。

注意：如果文件已经存在，`>` 会覆盖原内容。

```bash
echo "第一行" > test.txt
```

---

## 2. 追加输出 `>>`

把输出追加到文件末尾，不覆盖原内容。

```bash
echo "第二行" >> test.txt
```

此时 `test.txt` 中可能是：

```text
第一行
第二行
```

常用于持续记录日志：

```bash
date >> run.log
```

---

## 3. 输入重定向 `<`

让命令从文件中读取输入，而不是等待键盘输入。

```bash
wc -l < test.txt
```

含义是：读取 `test.txt`，统计其中的行数。

下面两个命令结果接近：

```bash
wc -l test.txt
wc -l < test.txt
```

区别是：

```bash
wc -l test.txt
```

通常会同时显示文件名。

而：

```bash
wc -l < test.txt
```

只显示统计结果，因为 `wc` 不知道输入来自哪个文件。

---

## 4. 错误输出重定向 `2>`

标准错误的编号是 `2`。

```bash
ls /not-exist 2> error.log
```

由于目录不存在，错误信息会写入 `error.log`，而不是显示在屏幕上。

追加错误日志：

```bash
ls /not-exist 2>> error.log
```

---

## 5. 同时重定向正常输出和错误输出

### 分别写入不同文件

```bash
command > output.log 2> error.log
```

其中：

```text
> output.log
```

等价于：

```text
1> output.log
```

完整写法是：

```bash
command 1> output.log 2> error.log
```

### 写入同一个文件

常用写法：

```bash
command > all.log 2>&1
```

含义是：

1. 先把标准输出写入 `all.log`
2. 再让标准错误跟随标准输出

在 Bash 中，也可以简写为：

```bash
command &> all.log
```

追加到同一个文件：

```bash
command >> all.log 2>&1
```

或者：

```bash
command &>> all.log
```

---

## 6. `2>&1` 中各符号的含义

```bash
2>&1
```

可以拆开理解：

* `2`：标准错误
* `>`：重定向
* `&1`：标准输出这个通道，而不是名为 `1` 的文件

因此，`2>&1` 的意思是：

> 让标准错误发送到标准输出当前所指向的位置。

### 顺序非常重要

```bash
command > all.log 2>&1
```

正常输出和错误输出都会进入 `all.log`。

但下面这条命令不同：

```bash
command 2>&1 > all.log
```

执行顺序是：

1. 标准错误先指向当前标准输出，也就是屏幕
2. 标准输出再改到 `all.log`

最终结果：

* 正常输出进入文件
* 错误输出仍然显示在屏幕上

---

## 7. 管道 `|`

管道把前一个命令的标准输出，作为后一个命令的标准输入。

```bash
ls -l | grep ".log"
```

执行过程可以理解为：

```text
ls -l 的输出 → grep 的输入
```

例如查看包含 `root` 的进程：

```bash
ps aux | grep root
```

统计当前目录文件数量：

```bash
ls | wc -l
```

管道默认只传递标准输出，不传递标准错误。

如果希望错误信息也进入管道，可以使用：

```bash
command 2>&1 | grep "关键字"
```

Bash 还支持：

```bash
command |& grep "关键字"
```

---

## 8. `/dev/null`

`/dev/null` 是一个特殊设备，可以理解为“黑洞”。写入其中的内容会被直接丢弃。

忽略正常输出：

```bash
command > /dev/null
```

忽略错误输出：

```bash
command 2> /dev/null
```

忽略全部输出：

```bash
command > /dev/null 2>&1
```

例如安静地检查某个主机是否可达：

```bash
ping -c 1 192.168.1.1 > /dev/null 2>&1
```

---

## 9. `tee`：一边显示，一边保存

普通重定向后，输出通常不会显示在终端。

`tee` 可以同时把内容显示在屏幕上，并写入文件：

```bash
ls -l | tee files.txt
```

追加写入：

```bash
ls -l | tee -a files.txt
```

它常用于观察程序运行过程，同时保存日志：

```bash
./run.sh 2>&1 | tee run.log
```

---

## 10. Here Document：输入多行内容

使用 `<<` 可以向命令提供多行输入。

```bash
cat << EOF
第一行
第二行
第三行
EOF
```

也可以写入文件：

```bash
cat > config.txt << EOF
server=192.168.1.10
port=8080
mode=production
EOF
```

`EOF` 只是结束标记，也可以换成其他单词：

```bash
cat << END
hello
Linux
END
```

如果不希望其中的变量被展开，可以给结束标记加引号：

```bash
name="Linux"

cat << 'EOF'
$name
EOF
```

输出为：

```text
$name
```

而不是：

```text
Linux
```

---

## 11. Here String：输入一行字符串

使用 `<<<` 把一个字符串作为命令输入：

```bash
wc -w <<< "hello Linux world"
```

输出：

```text
3
```

也可以处理变量：

```bash
text="apple banana orange"
grep "banana" <<< "$text"
```

---

## 12. 自定义文件描述符

除了 `0、1、2`，还可以使用其他编号，例如 `3`。

```bash
exec 3> custom.log
echo "写入自定义通道" >&3
exec 3>&-
```

含义是：

1. 打开文件描述符 `3`，指向 `custom.log`
2. 把内容写入文件描述符 `3`
3. 关闭文件描述符 `3`

这类写法常见于复杂 Shell 脚本。

---

## 常用写法汇总

| 写法                         | 作用                |
| -------------------------- | ----------------- |
| `command > file`           | 覆盖保存正常输出          |
| `command >> file`          | 追加保存正常输出          |
| `command < file`           | 从文件读取输入           |
| `command 2> file`          | 覆盖保存错误输出          |
| `command 2>> file`         | 追加保存错误输出          |
| `command > file 2>&1`      | 正常和错误输出写入同一文件     |
| `command &> file`          | Bash 中同时保存正常和错误输出 |
| `command \| other`         | 把前一命令输出交给后一命令     |
| `command \|& other`        | 正常和错误输出都交给后一命令    |
| `command > /dev/null 2>&1` | 丢弃全部输出            |
| `command \| tee file`      | 屏幕显示并保存文件         |
| `command << EOF`           | 输入多行内容            |
| `command <<< "text"`       | 输入一行字符串           |

## 一个完整示例

```bash
find / -name "*.conf" > result.log 2> error.log
```

执行后：

* 找到的配置文件路径写入 `result.log`
* 权限不足等错误写入 `error.log`
* 屏幕上不显示这些内容

若要同时显示并保存：

```bash
find / -name "*.conf" 2>&1 | tee result.log
```

这就是 Linux 重定向的核心：**控制命令从哪里读数据，以及把结果发送到哪里。**