---
title: Linux Shell 函数
section: IT
category: Linux
---

# Linux Shell 函数

<img src="images/Linux.svg" width="300">

Linux Shell 中的**函数（Function）**是一段可以重复调用的 Shell 代码。使用函数可以避免重复编写相同的代码，使脚本更加清晰、易于维护。

---

## 一、函数定义

Shell 函数有两种常见写法。

### 写法一（推荐，兼容性最好）

```bash
function_name() {
    command1
    command2
}
```

例如：

```bash
hello() {
    echo "Hello, World!"
}

hello
```

输出：

```text
Hello, World!
```

---

### 写法二（Bash 支持）

```bash
function function_name {
    command1
    command2
}
```

例如：

```bash
function hello {
    echo "Hello, World!"
}

hello
```

> 推荐第一种写法，因为兼容 POSIX Shell。

---

## 二、函数参数

函数可以接收参数，与脚本参数使用方式相同。

例如：

```bash
greet() {
    echo "Hello, $1"
}

greet Tom
greet Alice
```

输出：

```text
Hello, Tom
Hello, Alice
```

函数内部可使用的参数：

| 参数   | 含义          |
| ---- | ----------- |
| `$1` | 第一个参数       |
| `$2` | 第二个参数       |
| `$3` | 第三个参数       |
| `$#` | 参数个数        |
| `$*` | 所有参数（一个字符串） |
| `$@` | 所有参数（推荐）    |

例如：

```bash
show_args() {
    echo "参数个数：$#"
    echo "所有参数：$@"
}

show_args a b c
```

输出：

```text
参数个数：3
所有参数：a b c
```

---

## 三、返回值

Shell 函数只能使用 `return` 返回**整数状态码**。

```bash
check_file() {
    if [ -f "$1" ]; then
        return 0
    else
        return 1
    fi
}

check_file test.txt

echo $?
```

输出：

```text
0
```

其中：

* `0`：成功
* 非 `0`：失败

> `return` 的返回值范围通常为 **0~255**。

---

## 四、返回字符串

如果需要返回字符串，一般使用**标准输出**。

例如：

```bash
get_date() {
    echo "$(date +%F)"
}

today=$(get_date)

echo "$today"
```

输出：

```text
2026-07-27
```

这是 Shell 中最常见的“返回值”方式。

---

## 五、局部变量

默认情况下，函数中的变量都是**全局变量**。

例如：

```bash
test() {
    name="Tom"
}

test

echo "$name"
```

输出：

```text
Tom
```

如果希望变量只在函数内有效，应使用 `local`：

```bash
test() {
    local name="Tom"
    echo "$name"
}

test

echo "$name"
```

输出：

```text
Tom

```

第二次输出为空。

> `local` 是 Bash、Zsh 等支持的扩展，不属于 POSIX 标准。

---

## 六、函数调用其他函数

函数之间可以互相调用。

```bash
hello() {
    echo "Hello"
}

welcome() {
    hello
    echo "Welcome!"
}

welcome
```

输出：

```text
Hello
Welcome!
```

---

## 七、递归函数

Shell 支持递归，但效率较低，不适合复杂计算。

例如计算阶乘：

```bash
factorial() {
    local n=$1

    if [ "$n" -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $((n - 1)))
        echo $((n * prev))
    fi
}

factorial 5
```

输出：

```text
120
```

---

## 八、获取函数返回状态

函数执行完成后，可以通过 `$?` 获取返回状态。

```bash
say() {
    echo "Hello"
    return 100
}

say

echo $?
```

输出：

```text
Hello
100
```

---

## 九、删除函数

使用 `unset -f` 删除函数。

```bash
hello() {
    echo Hello
}

unset -f hello
```

再次调用：

```bash
hello
```

输出：

```text
bash: hello: command not found
```

---

## 十、查看函数

查看某个函数：

```bash
declare -f hello
```

查看所有函数：

```bash
declare -F
```

---

## 十一、实际应用示例

### 示例1：日志函数

```bash
log() {
    echo "[$(date '+%F %T')] $*"
}

log "程序启动"
log "开始备份"
```

输出：

```text
[2026-07-27 09:30:00] 程序启动
[2026-07-27 09:30:01] 开始备份
```

---

### 示例2：检查命令是否存在

```bash
check_cmd() {
    command -v "$1" >/dev/null 2>&1
}

if check_cmd git; then
    echo "Git 已安装"
else
    echo "Git 未安装"
fi
```

---

### 示例3：统一错误处理

```bash
die() {
    echo "Error: $*" >&2
    exit 1
}

[ -f config.ini ] || die "配置文件不存在"
```

---

## 十二、编写函数的最佳实践

1. **函数职责单一**：一个函数只完成一项任务。
2. **优先使用 `local`**：避免污染全局变量。
3. **参数加双引号**：例如 `"$1"`、`"$@"`，防止空格或特殊字符导致问题。
4. **返回状态码，数据用标准输出**：使用 `return` 表示执行是否成功，使用 `echo` 输出需要返回的数据。
5. **函数名采用小写加下划线**：如 `check_file`、`backup_data`，增强可读性。
6. **充分复用函数**：将重复逻辑封装为函数，减少代码冗余。

## 总结

| 功能    | 用法                             |
| ----- | ------------------------------ |
| 定义函数  | `func() { ... }`               |
| 调用函数  | `func arg1 arg2`               |
| 获取参数  | `$1`、`$2`、`$@`、`$#`            |
| 返回状态  | `return 0`、`return 1`          |
| 返回数据  | `echo` + 命令替换 `$(func)`        |
| 局部变量  | `local var=value`              |
| 获取返回码 | `$?`                           |
| 删除函数  | `unset -f func`                |
| 查看函数  | `declare -f func`、`declare -F` |

对于 Bash 脚本开发来说，函数是组织代码最重要的工具之一。掌握函数的定义、参数传递、返回值、作用域以及良好的封装习惯，可以显著提高脚本的可维护性和复用性。