---
title: Linux Shell 运算符
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoEX3
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/173'
---

# Linux Shell 运算符

<img src="images/Linux.svg" width="300">

# Linux Shell 常用运算符

Shell 运算符主要用于命令连接、条件判断、数值计算、字符串比较和文件检测。下面以 Bash 为例说明。

## 一、命令连接运算符

### 1. `;` 顺序执行

不管前一条命令是否成功，都会继续执行后一条命令。

```bash
mkdir test; cd test
```

### 2. `&&` 前一条成功才执行后一条

```bash
mkdir test && cd test
```

只有目录创建成功，才会进入目录。

### 3. `||` 前一条失败才执行后一条

```bash
cd /data || echo "目录不存在"
```

### 4. `&` 后台运行

```bash
python app.py &
```

命令会在后台运行，终端仍可继续输入其他命令。

### 5. `|` 管道

把前一条命令的输出，交给后一条命令处理。

```bash
ps aux | grep nginx
```

可以把管道理解为一根水管：前一个命令产生数据，后一个命令接收并处理数据。

---

## 二、重定向运算符

### 1. `>` 覆盖写入文件

```bash
echo "hello" > test.txt
```

文件原有内容会被覆盖。

### 2. `>>` 追加写入文件

```bash
echo "world" >> test.txt
```

新内容会追加到文件末尾。

### 3. `<` 从文件读取输入

```bash
wc -l < test.txt
```

### 4. `2>` 重定向错误信息

```bash
ls /not_exists 2> error.log
```

### 5. `2>&1` 合并标准错误和标准输出

```bash
command > output.log 2>&1
```

在 Bash 中，也可以写成：

```bash
command &> output.log
```

---

## 三、算术运算符

Shell 算术运算通常写在 `$(( ))` 中。

```bash
a=10
b=3

echo $((a + b))
echo $((a - b))
echo $((a * b))
echo $((a / b))
echo $((a % b))
```

常见运算符如下：

| 运算符  | 含义   |
| ---- | ---- |
| `+`  | 加法   |
| `-`  | 减法   |
| `*`  | 乘法   |
| `/`  | 整数除法 |
| `%`  | 取余   |
| `**` | 幂运算  |
| `++` | 自增   |
| `--` | 自减   |

示例：

```bash
num=5
((num++))
echo "$num"
```

输出：

```text
6
```

注意：Shell 默认进行整数运算，不能直接处理普通小数。

---

## 四、数值比较运算符

在传统的 `[ ]` 条件判断中，数值比较不能直接使用 `>` 和 `<`，通常使用以下运算符：

| 运算符   | 含义   |
| ----- | ---- |
| `-eq` | 等于   |
| `-ne` | 不等于  |
| `-gt` | 大于   |
| `-ge` | 大于等于 |
| `-lt` | 小于   |
| `-le` | 小于等于 |

示例：

```bash
a=10
b=5

if [ "$a" -gt "$b" ]; then
    echo "a 大于 b"
fi
```

在 `(( ))` 中，可以直接使用常见数学比较符号：

```bash
if (( a > b )); then
    echo "a 大于 b"
fi
```

数值判断时，`(( ))` 通常更直观。

---

## 五、字符串比较运算符

| 运算符  | 含义                |
| ---- | ----------------- |
| `=`  | 字符串相等             |
| `==` | 字符串相等，常用于 `[[ ]]` |
| `!=` | 字符串不相等            |
| `-z` | 字符串长度为零           |
| `-n` | 字符串长度不为零          |

示例：

```bash
name="Linux"

if [ "$name" = "Linux" ]; then
    echo "字符串相等"
fi
```

检查空字符串：

```bash
value=""

if [ -z "$value" ]; then
    echo "value 为空"
fi
```

建议变量两侧加双引号，避免变量为空或包含空格时出现判断错误。

---

## 六、逻辑运算符

### 在 `[ ]` 中

| 运算符  | 含义      |
| ---- | ------- |
| `!`  | 逻辑非     |
| `-a` | 逻辑与，不推荐 |
| `-o` | 逻辑或，不推荐 |

传统写法：

```bash
if [ "$a" -gt 0 ] && [ "$b" -gt 0 ]; then
    echo "两个数都大于 0"
fi
```

### 在 `[[ ]]` 中

可以直接使用：

```bash
if [[ "$name" == "Linux" && "$age" -ge 18 ]]; then
    echo "条件成立"
fi
```

常见逻辑符号：

| 运算符  | 含义  |   |     |
| ---- | --- | - | --- |
| `&&` | 逻辑与 |   |     |
| `    |     | ` | 逻辑或 |
| `!`  | 逻辑非 |   |     |

---

## 七、文件判断运算符

| 运算符  | 含义      |
| ---- | ------- |
| `-e` | 文件或目录存在 |
| `-f` | 是普通文件   |
| `-d` | 是目录     |
| `-r` | 可读      |
| `-w` | 可写      |
| `-x` | 可执行     |
| `-s` | 文件大小不为零 |
| `-L` | 是符号链接   |

示例：

```bash
file="/etc/passwd"

if [ -f "$file" ]; then
    echo "文件存在"
fi
```

判断目录：

```bash
if [ -d "/var/log" ]; then
    echo "目录存在"
fi
```

---

## 八、模式匹配运算符

在 `[[ ]]` 中，`==` 可以进行通配符匹配。

```bash
filename="report.txt"

if [[ "$filename" == *.txt ]]; then
    echo "这是文本文件"
fi
```

正则表达式匹配使用 `=~`：

```bash
value="12345"

if [[ "$value" =~ ^[0-9]+$ ]]; then
    echo "全部是数字"
fi
```

注意：使用 `=~` 时，正则表达式通常不要加引号。

---

## 九、变量默认值运算符

Shell 还支持一些非常实用的变量扩展运算符。

### 设置默认值

```bash
echo "${name:-默认名称}"
```

当 `name` 未定义或为空时，使用“默认名称”。

### 赋予默认值

```bash
echo "${name:=默认名称}"
```

不仅使用默认值，还会把默认值赋给变量。

### 变量为空时报错

```bash
echo "${name:?name 不能为空}"
```

### 变量有值时使用替代内容

```bash
echo "${name:+变量已有值}"
```

---

## 十、`[ ]`、`[[ ]]` 和 `(( ))` 的区别

### `[ ]`

传统条件判断命令，兼容性较好。

```bash
if [ "$name" = "Linux" ]; then
    echo "匹配"
fi
```

注意：`[` 和 `]` 两侧必须有空格。

### `[[ ]]`

Bash 提供的增强条件判断，支持逻辑组合、通配符和正则表达式。

```bash
if [[ "$name" == L* ]]; then
    echo "以 L 开头"
fi
```

编写 Bash 脚本时，字符串和复杂条件判断通常优先使用 `[[ ]]`。

### `(( ))`

专门用于整数计算和数值判断。

```bash
if (( a >= 10 )); then
    echo "a 大于等于 10"
fi
```

---

## 综合示例

```bash
#!/bin/bash

file="/var/log/syslog"
count=10

if [[ -f "$file" && -r "$file" ]]; then
    echo "文件存在且可读"

    if (( count > 5 )); then
        grep "error" "$file" > error.log 2>&1
    fi
else
    echo "文件不存在或不可读"
fi
```

实际使用时，可以简单记成：

* 命令之间用 `&&`、`||`、`|`。
* 字符串和文件判断用 `[[ ]]`。
* 整数计算和比较用 `(( ))`。
* 输出到文件用 `>` 或 `>>`。
