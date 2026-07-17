---
title: Linux Shell编程基础
section: IT
category: Linux
---

# Linux Shell编程基础

<img src="images/Linux.svg" width="300">

## 一、Shell 是什么

Shell 是用户与 Linux 系统之间的“命令解释器”。用户输入命令，Shell 负责解析命令，再调用操作系统中的程序执行。

例如：

```bash
ls -l /home
```

Shell 会解析这条命令，然后启动 `ls` 程序，显示 `/home` 目录中的文件。

Shell 不仅可以逐条执行命令，还能把一组命令写入文件，形成 **Shell 脚本**，用于自动完成重复性工作。

常见 Shell 包括：

* `sh`：传统 Bourne Shell，很多系统脚本以它为兼容基础。
* `bash`：Linux 中最常用，功能丰富。
* `dash`：体积小、启动快，常用于执行系统脚本。
* `zsh`：交互功能较强，常用于个人终端环境。
* `ksh`：较早支持高级脚本功能的 Shell。

POSIX 标准规定了一套通用 Shell 语法。追求跨系统兼容时，可以使用 `#!/bin/sh`；明确需要 Bash 特性时，应使用 `#!/usr/bin/env bash`。([pubs.opengroup.org][1])

---

## 二、第一个 Shell 脚本

新建文件：

```bash
vi hello.sh
```

输入：

```bash
#!/usr/bin/env bash

echo "Hello, Linux!"
```

第一行称为 **Shebang**，用于指定脚本由哪个解释器执行。

增加执行权限：

```bash
chmod +x hello.sh
```

运行脚本：

```bash
./hello.sh
```

也可以直接使用 Bash 执行：

```bash
bash hello.sh
```

两种方式有一点区别：

```bash
./hello.sh
```

按照脚本第一行指定的解释器执行。

```bash
bash hello.sh
```

直接使用当前调用的 Bash 执行，不依赖第一行。

---

## 三、注释与命令输出

### 1. 单行注释

以 `#` 开头：

```bash
# 这是注释
echo "开始执行"
```

Shebang 虽然也以 `#` 开头，但它不是普通注释。

### 2. 输出内容

```bash
echo "系统启动成功"
```

对于格式化输出，推荐使用 `printf`：

```bash
printf "用户名：%s\n" "admin"
printf "数量：%d\n" 10
```

与 `echo` 相比，`printf` 的输出格式更明确，跨不同 Shell 时也更稳定。

---

## 四、变量

### 1. 定义变量

```bash
name="Linux服务器"
count=10
```

注意，等号两边不能有空格。

错误写法：

```bash
name = "Linux服务器"
```

Shell 会把 `name` 当成命令执行。

### 2. 使用变量

```bash
name="Linux服务器"

echo "$name"
echo "当前设备：$name"
echo "当前设备：${name}"
```

在字符串中，推荐使用：

```bash
"${name}"
```

大括号能够明确变量名的边界：

```bash
file="report"

echo "${file}_2026.txt"
```

### 3. 只读变量

```bash
readonly system_name="生产管理系统"
```

之后不能再修改：

```bash
system_name="其他系统"
```

### 4. 删除变量

```bash
unset name
```

---

## 五、变量引用为什么要加双引号

假设：

```bash
file="生产 报告.txt"
```

不加双引号：

```bash
rm $file
```

Shell 可能将它拆成两个参数：

```text
生产
报告.txt
```

正确写法：

```bash
rm -- "$file"
```

因此，普通变量展开应尽量使用双引号：

```bash
echo "$name"
cp "$source_file" "$target_dir"
```

这是 Shell 编程中非常重要的习惯。

---

## 六、读取用户输入

```bash
#!/usr/bin/env bash

printf "请输入用户名："
read -r username

printf "你好，%s\n" "$username"
```

`-r` 表示按原样读取反斜杠，通常推荐使用。

一次读取多个值：

```bash
read -r name age
echo "姓名：$name"
echo "年龄：$age"
```

---

## 七、脚本参数

运行脚本时，可以传入参数：

```bash
./user.sh admin 20
```

脚本中可以通过特殊变量获取：

```bash
#!/usr/bin/env bash

echo "脚本名称：$0"
echo "第一个参数：$1"
echo "第二个参数：$2"
echo "参数数量：$#"
echo "所有参数：$*"
```

常用特殊变量：

| 变量        | 含义            |
| --------- | ------------- |
| `$0`      | 脚本名称          |
| `$1`～`$9` | 第1～第9个参数      |
| `$#`      | 参数数量          |
| `"$@"`    | 所有参数，每个参数保持独立 |
| `$?`      | 上一条命令的退出状态    |
| `$$`      | 当前 Shell 的进程号 |

处理所有参数时，推荐使用：

```bash
for arg in "$@"; do
    echo "$arg"
done
```

不要轻易使用未加引号的 `$*`，因为它可能破坏原来的参数边界。

---

## 八、命令替换

将命令结果保存到变量：

```bash
current_date=$(date "+%Y-%m-%d")
echo "$current_date"
```

获取主机名：

```bash
hostname_value=$(hostname)
echo "当前主机：$hostname_value"
```

推荐使用：

```bash
$(command)
```

不推荐旧式写法：

```bash
`command`
```

因为 `$()` 更容易阅读，也方便嵌套。

---

## 九、整数运算

### 1. 算术展开

```bash
a=10
b=20

sum=$((a + b))
echo "$sum"
```

常用运算符：

```bash
$((a + b))
$((a - b))
$((a * b))
$((a / b))
$((a % b))
```

Shell 原生算术通常处理整数，不适合直接进行高精度小数计算。

### 2. 自增

```bash
count=0
count=$((count + 1))
```

在 Bash 中也可以写：

```bash
((count++))
```

但在启用 `set -e` 时，`((count++))` 第一次执行可能返回非零状态。更稳妥的写法是：

```bash
((count += 1))
```

---

## 十、条件判断

### 1. 基本结构

```bash
if 条件; then
    命令
fi
```

例如：

```bash
age=20

if [ "$age" -ge 18 ]; then
    echo "已成年"
fi
```

注意：`[` 和 `]` 本质上是命令，因此两边必须留空格。

错误：

```bash
if ["$age" -ge 18]; then
```

正确：

```bash
if [ "$age" -ge 18 ]; then
```

### 2. if、elif、else

```bash
score=85

if [ "$score" -ge 90 ]; then
    echo "优秀"
elif [ "$score" -ge 60 ]; then
    echo "合格"
else
    echo "不合格"
fi
```

### 3. 数值比较

| 表达式   | 含义    |
| ----- | ----- |
| `-eq` | 等于    |
| `-ne` | 不等于   |
| `-gt` | 大于    |
| `-ge` | 大于或等于 |
| `-lt` | 小于    |
| `-le` | 小于或等于 |

例如：

```bash
if [ "$count" -gt 10 ]; then
    echo "数量大于10"
fi
```

### 4. 字符串比较

```bash
if [ "$name" = "admin" ]; then
    echo "管理员用户"
fi
```

常用判断：

```bash
[ "$a" = "$b" ]       # 相等
[ "$a" != "$b" ]      # 不相等
[ -z "$a" ]           # 字符串为空
[ -n "$a" ]           # 字符串非空
```

### 5. 文件判断

```bash
[ -e "$file" ]    # 路径存在
[ -f "$file" ]    # 是普通文件
[ -d "$file" ]    # 是目录
[ -r "$file" ]    # 可读
[ -w "$file" ]    # 可写
[ -x "$file" ]    # 可执行
[ -s "$file" ]    # 文件大小大于0
```

示例：

```bash
config_file="/etc/hosts"

if [ -f "$config_file" ]; then
    echo "配置文件存在"
else
    echo "配置文件不存在"
fi
```

---

## 十一、`[ ]` 与 `[[ ]]` 的区别

`[ ]` 是传统测试语法，兼容性更好：

```bash
if [ "$name" = "admin" ]; then
    echo "匹配成功"
fi
```

`[[ ]]` 是 Bash、Ksh 等 Shell 提供的增强语法：

```bash
if [[ "$name" == admin* ]]; then
    echo "用户名以 admin 开头"
fi
```

在 Bash 脚本中，`[[ ]]` 对字符串判断更安全、功能也更强。但它不属于所有 POSIX Shell 都支持的通用语法。

因此：

* `#!/bin/sh` 脚本使用 `[ ]`。
* `#!/usr/bin/env bash` 脚本可以使用 `[[ ]]`。

---

## 十二、逻辑运算

### 1. 命令成功后继续执行

```bash
mkdir -p backup && echo "目录创建成功"
```

只有前面的命令成功，才执行后面的命令。

### 2. 命令失败后执行

```bash
cp config.ini /backup/ || echo "复制失败"
```

只有前面的命令失败，才执行后面的命令。

### 3. 组合条件

Bash 中可以写：

```bash
if [[ "$age" -ge 18 && "$status" == "active" ]]; then
    echo "条件满足"
fi
```

---

## 十三、case 分支

当一个变量可能有多个值时，使用 `case` 比多层 `if` 更清楚。

```bash
#!/usr/bin/env bash

action="${1:-}"

case "$action" in
    start)
        echo "启动服务"
        ;;
    stop)
        echo "停止服务"
        ;;
    restart)
        echo "重启服务"
        ;;
    *)
        echo "用法：$0 {start|stop|restart}"
        exit 1
        ;;
esac
```

`${1:-}` 表示：

* 如果 `$1` 有值，使用 `$1`。
* 如果 `$1` 没有值，使用空字符串。

也可以设置默认值：

```bash
port="${1:-8080}"
```

---

## 十四、循环

### 1. for 循环

```bash
for name in server1 server2 server3; do
    echo "检查主机：$name"
done
```

遍历参数：

```bash
for arg in "$@"; do
    echo "参数：$arg"
done
```

遍历文件：

```bash
for file in /var/log/*.log; do
    [ -e "$file" ] || continue
    echo "日志文件：$file"
done
```

不要使用下面的方法处理文件名：

```bash
for file in $(ls); do
    echo "$file"
done
```

文件名中含空格、换行或通配符时，这种写法容易出错。

### 2. while 循环

```bash
count=1

while [ "$count" -le 5 ]; do
    echo "$count"
    count=$((count + 1))
done
```

逐行读取文件：

```bash
while IFS= read -r line; do
    echo "内容：$line"
done < input.txt
```

这是读取文本文件的常用写法。

### 3. 跳出循环

```bash
break
```

结束整个循环。

```bash
continue
```

跳过本次循环，进入下一次循环。

---

## 十五、函数

定义函数：

```bash
show_message() {
    echo "系统运行正常"
}
```

调用函数：

```bash
show_message
```

带参数的函数：

```bash
check_file() {
    local file="$1"

    if [ -f "$file" ]; then
        echo "文件存在：$file"
        return 0
    else
        echo "文件不存在：$file" >&2
        return 1
    fi
}

check_file "/etc/hosts"
```

函数内部：

* `$1` 表示函数的第一个参数。
* `local` 表示局部变量，是 Bash 的常用功能。
* `return` 返回状态码，而不是普通字符串。
* `0` 通常表示成功，非 `0` 表示失败。

Shell 函数需要返回数据时，可以输出内容：

```bash
get_date() {
    date "+%Y-%m-%d"
}

today=$(get_date)
echo "$today"
```

---

## 十六、退出状态

Linux 命令执行结束后会返回一个数字：

* `0`：执行成功。
* 非 `0`：执行失败或出现特定情况。

例如：

```bash
ls /etc/hosts
echo "$?"
```

脚本中可以主动退出：

```bash
if [ ! -f "/etc/hosts" ]; then
    echo "文件不存在" >&2
    exit 1
fi

exit 0
```

判断命令是否成功，通常直接判断命令，不必先读取 `$?`：

```bash
if ping -c 1 server1 >/dev/null 2>&1; then
    echo "主机可访问"
else
    echo "主机不可访问"
fi
```

---

## 十七、输入输出重定向

Linux 程序通常有三个标准通道：

| 编号  | 名称   | 作用     |
| --- | ---- | ------ |
| `0` | 标准输入 | 接收输入   |
| `1` | 标准输出 | 输出正常信息 |
| `2` | 标准错误 | 输出错误信息 |

### 1. 覆盖写入文件

```bash
echo "系统启动" > system.log
```

### 2. 追加写入文件

```bash
echo "系统停止" >> system.log
```

### 3. 错误输出到文件

```bash
command 2> error.log
```

### 4. 标准输出和错误同时保存

Bash 写法：

```bash
command > run.log 2>&1
```

较新的 Bash 也支持：

```bash
command &> run.log
```

前一种写法兼容性更好。

### 5. 丢弃输出

```bash
command >/dev/null 2>&1
```

`/dev/null` 像一个“黑洞”，写入其中的内容会被丢弃。

---

## 十八、管道

管道符 `|` 将前一个命令的输出交给后一个命令处理。

```bash
ps -ef | grep nginx
```

统计日志中的错误数量：

```bash
grep -i "error" application.log | wc -l
```

查看磁盘使用率：

```bash
df -h | grep -v tmpfs
```

Shell 的一个重要思想是：让多个简单命令像流水线一样配合工作。

---

## 十九、数组

数组主要是 Bash 功能，不属于通用 POSIX `sh` 语法。

```bash
servers=("server1" "server2" "server3")
```

读取数组：

```bash
echo "${servers[0]}"
```

遍历数组：

```bash
for server in "${servers[@]}"; do
    echo "$server"
done
```

获取数组长度：

```bash
echo "${#servers[@]}"
```

向数组增加元素：

```bash
servers+=("server4")
```

---

## 二十、常用文本处理命令

Shell 脚本经常与其他 Linux 命令配合。

### grep：查找文本

```bash
grep "ERROR" application.log
grep -i "error" application.log
grep -n "error" application.log
```

### sed：替换和处理文本

```bash
sed 's/old/new/g' input.txt
```

直接修改文件前应谨慎：

```bash
sed -i.bak 's/old/new/g' config.txt
```

这种写法会先保留一个 `.bak` 备份。

### awk：按列处理数据

```bash
awk '{print $1, $3}' data.txt
```

例如查看磁盘分区和使用率：

```bash
df -P | awk 'NR > 1 {print $1, $5}'
```

### sort 和 uniq：排序与统计

```bash
sort users.txt | uniq
```

统计重复次数：

```bash
sort users.txt | uniq -c
```

---

## 二十一、一个完整示例：检查磁盘使用率

```bash
#!/usr/bin/env bash

set -u

threshold="${1:-80}"

if ! [[ "$threshold" =~ ^[0-9]+$ ]]; then
    echo "错误：阈值必须是整数。" >&2
    exit 1
fi

df -P | awk -v threshold="$threshold" '
NR > 1 {
    usage = $5
    gsub(/%/, "", usage)

    if (usage >= threshold) {
        printf "告警：文件系统 %s 使用率为 %s%%，挂载点为 %s\n",
               $1, usage, $6
    }
}
'
```

运行：

```bash
chmod +x check_disk.sh
./check_disk.sh 85
```

不传参数时，默认阈值是 `80`：

```bash
./check_disk.sh
```

这个脚本包含了几个基础知识：

* 参数与默认值。
* 参数合法性检查。
* 管道和 `awk`。
* 条件判断。
* 错误输出。
* 退出状态。

---

## 二十二、脚本安全与规范

### 1. 使用严格模式时要理解其影响

很多 Bash 脚本会使用：

```bash
set -Eeuo pipefail
```

含义大致如下：

```bash
set -e
```

命令失败时尝试终止脚本。

```bash
set -u
```

使用未定义变量时报错。

```bash
set -o pipefail
```

管道中任意命令失败，整个管道返回失败。

```bash
set -E
```

让 `ERR` 陷阱在部分函数和子 Shell 环境中继承。

严格模式有助于发现问题，但 `set -e` 存在不少特殊规则，不能简单理解成“任何命令失败都必然退出”。正式脚本中应结合明确的错误判断使用。

### 2. 检查必要命令

```bash
if ! command -v curl >/dev/null 2>&1; then
    echo "错误：系统未安装 curl。" >&2
    exit 1
fi
```

### 3. 检查参数

```bash
if [ "$#" -ne 1 ]; then
    echo "用法：$0 <配置文件>" >&2
    exit 1
fi
```

### 4. 创建临时文件

不建议直接使用固定临时文件名：

```bash
temp_file="/tmp/data.tmp"
```

推荐：

```bash
temp_file=$(mktemp) || exit 1
```

脚本退出时删除：

```bash
trap 'rm -f "$temp_file"' EXIT
```

### 5. 谨慎执行删除操作

```bash
rm -rf "$target_dir"
```

执行前至少检查变量是否为空：

```bash
if [ -z "${target_dir:-}" ] || [ "$target_dir" = "/" ]; then
    echo "拒绝删除危险路径。" >&2
    exit 1
fi

rm -rf -- "$target_dir"
```

### 6. 不要直接执行不可信字符串

下面的代码风险较高：

```bash
eval "$user_input"
```

`eval` 会把字符串重新当作 Shell 代码执行。除非完全理解输入内容，否则应避免使用。

---

## 二十三、语法检查与调试

### 1. 只检查语法

```bash
bash -n script.sh
```

它不会真正执行脚本。

### 2. 显示执行过程

```bash
bash -x script.sh
```

也可以在脚本中临时开启：

```bash
set -x
```

关闭：

```bash
set +x
```

注意不要在调试输出中泄露密码、令牌等敏感数据。

### 3. 使用 ShellCheck

ShellCheck 是专门检查 Shell 脚本问题的静态分析工具，可以发现变量未加引号、错误判断、无效语法和其他常见问题。([ShellCheck][2])

使用方式：

```bash
shellcheck script.sh
```

建议形成固定流程：

```bash
bash -n script.sh
shellcheck script.sh
./script.sh
```

---

## 二十四、推荐的脚本结构

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

readonly SCRIPT_NAME=$(basename "$0")

log() {
    printf '[%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$*"
}

error() {
    printf '[ERROR] %s\n' "$*" >&2
}

usage() {
    printf '用法：%s <文件路径>\n' "$SCRIPT_NAME"
}

main() {
    if [ "$#" -ne 1 ]; then
        usage
        return 1
    fi

    local file="$1"

    if [ ! -f "$file" ]; then
        error "文件不存在：$file"
        return 1
    fi

    log "开始处理文件：$file"

    wc -l < "$file"

    log "处理完成"
}

main "$@"
```

这种结构有几个好处：

* 主流程集中在 `main` 函数中。
* 日志和错误输出相互分开。
* 参数检查清楚。
* 函数变量使用 `local`。
* 变量使用双引号保护。
* 脚本最后通过 `main "$@"` 传递全部参数。

---

## 二十五、学习重点

入门阶段建议依次掌握：

1. Linux 常用命令、路径和权限。
2. 变量、参数和命令替换。
3. `if`、`case` 条件判断。
4. `for`、`while` 循环。
5. 函数和退出状态。
6. 管道与输入输出重定向。
7. `grep`、`sed`、`awk` 文本处理。
8. 参数校验、错误处理和日志。
9. `bash -n`、`bash -x` 和 ShellCheck。
10. 编写备份、巡检、日志分析等实际脚本。

GNU Bash 手册是 Bash 行为的重要参考；需要编写跨平台脚本时，应同时参考 POSIX Shell Command Language。([GNU][3])

Shell 编程不适合用来建设复杂的大型业务系统，但特别适合做系统管理、任务调度、文件处理、日志分析和运维自动化。可以把它理解为 Linux 管理中的“胶水”：单个命令能力有限，组合起来却能快速完成完整任务。

[1]: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/V3_chap02.html "2. Shell Command Language"
[2]: https://www.shellcheck.net/ "ShellCheck – shell script analysis tool"
[3]: https://www.gnu.org/s/bash/manual/bash.html "Bash Reference Manual"