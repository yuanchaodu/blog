---
title: Linux Shell 变量
section: IT
category: Linux
---

# Linux Shell 变量

<img src="images/Linux.svg" width="300">

## Linux Shell 变量

Shell 变量可以理解为：**给一段数据起一个名字，后面通过名字使用这段数据。**

例如，把用户名保存到变量中：

```bash
username="Nick"
echo "$username"
```

输出：

```text
Nick
```

---

## 一、变量的定义

基本格式：

```bash
变量名=变量值
```

例如：

```bash
name="Nick"
age=18
path="/home/nick"
```

需要注意：**等号两边不能有空格。**

错误写法：

```bash
name = "Nick"
```

正确写法：

```bash
name="Nick"
```

变量名通常只能包含：

* 英文字母
* 数字
* 下划线 `_`

并且不能以数字开头。

```bash
user_name="Nick"
server01="Linux"
```

---

## 二、变量的使用

使用变量时，在变量名前加 `$`：

```bash
name="Nick"
echo $name
```

更推荐加上双引号：

```bash
echo "$name"
```

双引号可以避免变量值中包含空格或特殊字符时出现问题。

例如：

```bash
message="Hello Linux"
echo "$message"
```

---

## 三、使用花括号

当变量名后面紧跟其他字符时，可以使用 `${变量名}`。

```bash
file="report"
echo "${file}.txt"
```

输出：

```text
report.txt
```

如果写成：

```bash
echo "$file.txt"
```

Shell 可能会把 `file.txt` 错误地理解为变量名的一部分。

---

## 四、变量的主要类型

### 1. 普通变量

只在当前 Shell 中有效。

```bash
name="Nick"
```

启动一个新的子 Shell 后，这个变量通常不会自动传过去。

---

### 2. 环境变量

环境变量可以传递给当前 Shell 启动的程序或子 Shell。

使用 `export` 定义：

```bash
export APP_HOME="/opt/app"
```

也可以分两步：

```bash
APP_HOME="/opt/app"
export APP_HOME
```

查看变量：

```bash
echo "$APP_HOME"
```

常见环境变量包括：

```bash
echo "$HOME"
echo "$PATH"
echo "$USER"
echo "$SHELL"
echo "$PWD"
```

其中：

| 变量      | 含义           |
| ------- | ------------ |
| `HOME`  | 当前用户的家目录     |
| `PATH`  | 命令搜索路径       |
| `USER`  | 当前用户名        |
| `SHELL` | 当前用户默认 Shell |
| `PWD`   | 当前工作目录       |

查看全部环境变量：

```bash
env
```

或者：

```bash
printenv
```

---

### 3. Shell 特殊变量

Shell 会自动提供一些特殊变量。

| 变量        | 含义            |
| --------- | ------------- |
| `$0`      | 当前脚本名称        |
| `$1`～`$9` | 脚本的位置参数       |
| `$#`      | 参数数量          |
| `$@`      | 所有参数          |
| `$?`      | 上一条命令的退出状态    |
| `$$`      | 当前 Shell 的进程号 |
| `$!`      | 最近一个后台进程的进程号  |

示例脚本 `test.sh`：

```bash
#!/bin/bash

echo "脚本名称：$0"
echo "第一个参数：$1"
echo "第二个参数：$2"
echo "参数数量：$#"
echo "所有参数：$@"
```

执行：

```bash
bash test.sh apple banana
```

输出类似：

```text
脚本名称：test.sh
第一个参数：apple
第二个参数：banana
参数数量：2
所有参数：apple banana
```

---

## 五、命令结果赋值给变量

可以把一条命令的输出保存到变量中：

```bash
current_date=$(date)
echo "$current_date"
```

获取当前目录：

```bash
current_dir=$(pwd)
echo "$current_dir"
```

旧式写法使用反引号：

```bash
current_date=`date`
```

推荐使用 `$(命令)`，因为它更清晰，也更方便嵌套。

---

## 六、变量中的引号

### 双引号

双引号中的变量会被解析：

```bash
name="Nick"
echo "Hello, $name"
```

输出：

```text
Hello, Nick
```

### 单引号

单引号中的内容会原样输出：

```bash
name="Nick"
echo 'Hello, $name'
```

输出：

```text
Hello, $name
```

可以把它理解为：

* 双引号：“里面的变量需要替换”
* 单引号：“里面写什么就显示什么”

---

## 七、只读变量

使用 `readonly` 可以防止变量被修改：

```bash
readonly company="ABC"
```

再次赋值会报错：

```bash
company="XYZ"
```

也可以这样写：

```bash
company="ABC"
readonly company
```

---

## 八、删除变量

使用 `unset` 删除变量：

```bash
name="Nick"
unset name
echo "$name"
```

删除后，变量值为空。

只读变量不能通过 `unset` 删除。

---

## 九、设置默认值

当变量为空或未定义时，可以使用默认值：

```bash
echo "${username:-guest}"
```

如果 `username` 没有设置，输出：

```text
guest
```

常见写法：

```bash
${var:-default}
```

变量为空时，使用默认值，但不修改变量。

```bash
${var:=default}
```

变量为空时，使用默认值，同时把默认值赋给变量。

```bash
${var:?错误信息}
```

变量为空时，显示错误并终止脚本。

例如：

```bash
database="${DB_NAME:-testdb}"
echo "$database"
```

---

## 十、数值运算

Shell 变量默认按字符串处理。进行整数运算时，可以使用双括号：

```bash
a=10
b=20
result=$((a + b))

echo "$result"
```

输出：

```text
30
```

也可以直接修改变量：

```bash
count=1
count=$((count + 1))
```

或者：

```bash
((count++))
```

Shell 原生通常只支持整数运算，不直接支持小数运算。

---

## 十一、数组变量

Bash 支持数组：

```bash
servers=("server01" "server02" "server03")
```

读取第一个元素：

```bash
echo "${servers[0]}"
```

读取所有元素：

```bash
echo "${servers[@]}"
```

读取数组长度：

```bash
echo "${#servers[@]}"
```

遍历数组：

```bash
for server in "${servers[@]}"; do
    echo "$server"
done
```

---

## 十二、局部变量

在 Shell 函数中，可以使用 `local` 定义局部变量：

```bash
show_user() {
    local username="Nick"
    echo "$username"
}

show_user
```

`username` 主要在函数内部使用，不会轻易影响函数外部的同名变量。

---

## 十三、一个完整示例

```bash
#!/bin/bash

name="${1:-guest}"
current_time=$(date "+%Y-%m-%d %H:%M:%S")

echo "欢迎你，$name"
echo "当前时间：$current_time"
echo "当前目录：$PWD"

if [ -n "$HOME" ]; then
    echo "用户家目录：$HOME"
fi
```

执行：

```bash
bash welcome.sh Nick
```

可能输出：

```text
欢迎你，Nick
当前时间：2026-07-26 10:30:00
当前目录：/home/nick
用户家目录：/home/nick
```

## 核心要点

```bash
name="Nick"            # 定义变量
echo "$name"           # 使用变量
export name            # 导出为环境变量
unset name             # 删除变量
result=$(command)      # 保存命令输出
result=$((1 + 2))      # 整数运算
echo "${name:-guest}"  # 设置默认值
```

实际编写 Shell 脚本时，变量一般建议写成 `"$变量名"` 或 `"${变量名}"`，这样更安全，也能减少空格、通配符等带来的问题。