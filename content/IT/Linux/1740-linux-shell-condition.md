---
title: Linux 条件判断
section: IT
category: Linux
---

# Linux 条件判断

<img src="images/Linux.svg" width="300">

Linux Shell（通常指 Bash）中的**条件判断**主要用于 `if`、`while` 等流程控制语句，判断文件、字符串、数字或命令执行结果。最常见的有 `test` 命令、`[` `]`、`[[ ]]` 三种形式。

---

# 一、基本语法

## 1. if 语句

```bash
if 条件
then
    命令
fi
```

例如：

```bash
a=10

if [ $a -gt 5 ]
then
    echo "a 大于 5"
fi
```

也可以写成一行：

```bash
if [ $a -gt 5 ]; then
    echo "a 大于5"
fi
```

---

## 2. if...else

```bash
if [ 条件 ]
then
    命令1
else
    命令2
fi
```

例如：

```bash
score=80

if [ $score -ge 60 ]
then
    echo "及格"
else
    echo "不及格"
fi
```

---

## 3. if...elif...else

```bash
if [ 条件1 ]
then
    ...
elif [ 条件2 ]
then
    ...
else
    ...
fi
```

例如：

```bash
score=90

if [ $score -ge 90 ]
then
    echo "优秀"
elif [ $score -ge 60 ]
then
    echo "及格"
else
    echo "不及格"
fi
```

---

# 二、条件判断方式

## 1. 数值比较

| 操作符   | 含义       |
| ----- | -------- |
| `-eq` | 等于（==）   |
| `-ne` | 不等于（!=）  |
| `-gt` | 大于（>）    |
| `-lt` | 小于（<）    |
| `-ge` | 大于等于（>=） |
| `-le` | 小于等于（<=） |

例如：

```bash
a=5
b=10

if [ $a -lt $b ]
then
    echo "$a 小于 $b"
fi
```

---

# 三、字符串比较

| 操作符  | 含义    |
| ---- | ----- |
| `=`  | 相等    |
| `!=` | 不相等   |
| `-z` | 字符串为空 |
| `-n` | 字符串非空 |

例如：

```bash
name="Tom"

if [ "$name" = "Tom" ]
then
    echo "Hello Tom"
fi
```

判断空字符串：

```bash
str=""

if [ -z "$str" ]
then
    echo "字符串为空"
fi
```

判断非空：

```bash
if [ -n "$str" ]
then
    echo "字符串非空"
fi
```

**建议：变量最好加双引号。**

错误写法：

```bash
[ $name = Tom ]
```

正确写法：

```bash
[ "$name" = "Tom" ]
```

这样可以避免变量为空或包含空格时出错。

---

# 四、文件判断

Linux Shell 中经常需要判断文件是否存在。

| 判断        | 含义       |
| --------- | -------- |
| `-e file` | 文件存在     |
| `-f file` | 普通文件     |
| `-d file` | 目录       |
| `-r file` | 可读       |
| `-w file` | 可写       |
| `-x file` | 可执行      |
| `-s file` | 文件大小大于 0 |

例如：

```bash
if [ -f test.txt ]
then
    echo "文件存在"
fi
```

判断目录：

```bash
if [ -d /home ]
then
    echo "目录存在"
fi
```

---

# 五、逻辑运算

## 与（AND）

推荐：

```bash
if [[ $a -gt 0 && $b -gt 0 ]]
then
    echo "都大于0"
fi
```

传统写法：

```bash
if [ $a -gt 0 ] && [ $b -gt 0 ]
then
    echo "都大于0"
fi
```

---

## 或（OR）

```bash
if [[ $a -gt 0 || $b -gt 0 ]]
then
    echo "至少一个大于0"
fi
```

---

## 非（NOT）

```bash
if [ ! -f test.txt ]
then
    echo "文件不存在"
fi
```

---

# 六、`[` 与 `[[` 的区别

推荐在 Bash 中优先使用 `[[ ]]`。

| 特性        | `[ ]` | `[[ ]]`    |       |   |
| --------- | ----- | ---------- | ----- | - |
| POSIX 标准  | ✔     | ✘（Bash 扩展） |       |   |
| 支持 `&&`、` |       | `          | 不直接支持 | ✔ |
| 正则匹配      | ✘     | ✔          |       |   |
| 通配符匹配     | 有限制   | ✔          |       |   |
| 变量未加引号更安全 | ✘     | ✔          |       |   |

例如：

```bash
file="test.txt"

if [[ $file == *.txt ]]
then
    echo "txt 文件"
fi
```

正则匹配：

```bash
str="abc123"

if [[ $str =~ ^[a-z]+[0-9]+$ ]]
then
    echo "匹配成功"
fi
```

---

# 七、命令返回值判断

Linux 中**命令执行成功返回 0，失败返回非 0**。

例如：

```bash
if ping -c 1 192.168.1.1 >/dev/null
then
    echo "网络正常"
else
    echo "网络异常"
fi
```

或者判断上一条命令的退出状态：

```bash
cp file1 file2

if [ $? -eq 0 ]
then
    echo "复制成功"
fi
```

更推荐直接判断命令，而不是使用 `$?`：

```bash
if cp file1 file2
then
    echo "复制成功"
else
    echo "复制失败"
fi
```

---

# 八、case 条件判断

当需要进行多分支匹配时，`case` 比多个 `if...elif` 更清晰。

```bash
read -p "请输入(y/n): " answer

case $answer in
    y|Y)
        echo "Yes"
        ;;
    n|N)
        echo "No"
        ;;
    *)
        echo "输入错误"
        ;;
esac
```

---

# 九、综合示例

```bash
#!/bin/bash

read -p "请输入一个数字：" num

if [[ ! $num =~ ^[0-9]+$ ]]; then
    echo "请输入整数"
elif [ "$num" -gt 100 ]; then
    echo "大于100"
elif [ "$num" -eq 100 ]; then
    echo "等于100"
else
    echo "小于100"
fi
```

该示例综合了正则表达式、数值比较和多分支判断。

---

## 常用条件判断速查表

| 类型  | 操作符   | 示例                  |   |         |   |         |
| --- | ----- | ------------------- | - | ------- | - | ------- |
| 数值  | `-eq` | `[ "$a" -eq 10 ]`   |   |         |   |         |
| 数值  | `-ne` | `[ "$a" -ne 10 ]`   |   |         |   |         |
| 数值  | `-gt` | `[ "$a" -gt 10 ]`   |   |         |   |         |
| 数值  | `-lt` | `[ "$a" -lt 10 ]`   |   |         |   |         |
| 数值  | `-ge` | `[ "$a" -ge 10 ]`   |   |         |   |         |
| 数值  | `-le` | `[ "$a" -le 10 ]`   |   |         |   |         |
| 字符串 | `=`   | `[ "$s" = "abc" ]`  |   |         |   |         |
| 字符串 | `!=`  | `[ "$s" != "abc" ]` |   |         |   |         |
| 字符串 | `-z`  | `[ -z "$s" ]`       |   |         |   |         |
| 字符串 | `-n`  | `[ -n "$s" ]`       |   |         |   |         |
| 文件  | `-e`  | `[ -e file ]`       |   |         |   |         |
| 文件  | `-f`  | `[ -f file ]`       |   |         |   |         |
| 文件  | `-d`  | `[ -d dir ]`        |   |         |   |         |
| 文件  | `-r`  | `[ -r file ]`       |   |         |   |         |
| 文件  | `-w`  | `[ -w file ]`       |   |         |   |         |
| 文件  | `-x`  | `[ -x file ]`       |   |         |   |         |
| 文件  | `-s`  | `[ -s file ]`       |   |         |   |         |
| 逻辑  | `&&`  | `[[ 条件1 && 条件2 ]]`  |   |         |   |         |
| 逻辑  | `     |                     | ` | `[[ 条件1 |   | 条件2 ]]` |
| 逻辑  | `!`   | `[[ ! 条件 ]]`        |   |         |   |         |

**建议：**

* 编写 Bash 脚本时，优先使用 `[[ ... ]]`，功能更强且更安全。
* 变量引用尽量使用双引号（如 `"$var"`），可以避免变量为空或包含空格导致的语法错误。
* 判断命令执行结果时，优先直接以命令作为 `if` 条件，而不是依赖 `$?`，代码更简洁、可读性更好。