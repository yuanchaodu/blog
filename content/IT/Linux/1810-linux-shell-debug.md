---
title: Linux shell 调试
section: IT
category: Linux
---

# Linux shell 调试

<img src="images/Linux.svg" width="300">

Linux Shell 调试，主要是定位脚本中的语法错误、变量问题、命令执行失败和逻辑异常。下面是最常用的方法。

## 1. 检查语法错误

以 Bash 脚本为例：

```bash
bash -n script.sh
```

该命令只检查语法，不真正执行脚本。

常见语法问题包括：

```bash
if [ "$name" = "admin" ]; then
    echo "管理员"
fi
```

注意：

* `[` 和 `]` 两边必须有空格。
* `if` 必须以 `fi` 结束。
* `case` 必须以 `esac` 结束。
* 引号和括号要成对出现。

---

## 2. 显示脚本执行过程

```bash
bash -x script.sh
```

或者在脚本开头加入：

```bash
#!/bin/bash
set -x
```

执行时会显示每条命令展开后的内容，例如：

```text
+ name=Nick
+ echo 'Hello Nick'
Hello Nick
```

临时关闭跟踪：

```bash
set +x
```

只调试某一段代码：

```bash
set -x

result=$(some_command)
echo "$result"

set +x
```

---

## 3. 显示原始脚本命令

```bash
bash -v script.sh
```

区别如下：

* `-v`：显示脚本原始内容。
* `-x`：显示变量替换和命令展开后的内容。
* `-n`：只检查语法。

也可以组合使用：

```bash
bash -xv script.sh
```

---

## 4. 遇到错误立即退出

```bash
set -e
```

当某条命令执行失败时，脚本通常会立即停止。

更常见的严格模式是：

```bash
set -euo pipefail
```

含义如下：

```bash
set -e          # 命令失败时退出
set -u          # 使用未定义变量时报错
set -o pipefail # 管道中任意命令失败，整个管道视为失败
```

示例：

```bash
#!/bin/bash
set -euo pipefail

name="${1:-}"
echo "用户名：$name"
```

需要注意，`set -e` 在条件判断、循环和部分组合命令中有特殊行为，不应把它当作完整的异常处理机制。

---

## 5. 查看命令退出状态

Linux 命令执行后，可以通过 `$?` 查看退出码：

```bash
ls /not-exist
echo $?
```

一般情况下：

* `0`：执行成功。
* 非 `0`：执行失败。

实际脚本中，建议直接使用条件判断：

```bash
if cp source.txt target.txt; then
    echo "复制成功"
else
    echo "复制失败"
fi
```

相比下面的写法更稳妥：

```bash
cp source.txt target.txt
if [ $? -eq 0 ]; then
    echo "复制成功"
fi
```

---

## 6. 打印变量内容

最简单的方法是使用 `echo` 或 `printf`：

```bash
echo "name=[$name]"
printf 'name=<%s>\n' "$name"
```

用括号或尖括号包住变量，可以更容易发现空格和空值。

查看变量的准确形式：

```bash
declare -p name
```

查看数组：

```bash
declare -p files
```

查看字符串是否包含隐藏字符：

```bash
printf '%q\n' "$value"
```

例如：

```bash
value="hello world"
printf '%q\n' "$value"
```

输出：

```text
hello\ world
```

---

## 7. 定位错误发生的行号

可以定义错误捕获函数：

```bash
#!/bin/bash
set -E

trap 'echo "错误：第 $LINENO 行，命令：$BASH_COMMAND，退出码：$?" >&2' ERR

echo "开始"
cp /not-exist/file.txt /tmp/
echo "结束"
```

更完整的写法：

```bash
#!/bin/bash
set -Eeuo pipefail

trap 'rc=$?; echo "错误：脚本=${BASH_SOURCE[0]}，行号=$LINENO，命令=$BASH_COMMAND，退出码=$rc" >&2' ERR
```

其中：

* `$LINENO`：当前行号。
* `$BASH_COMMAND`：发生错误时执行的命令。
* `$?`：命令退出码。
* `${BASH_SOURCE[0]}`：脚本文件名。

---

## 8. 使用自定义日志函数

```bash
log() {
    printf '%s [%s] %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$1" \
        "$2"
}

log INFO "脚本开始执行"
log DEBUG "当前用户：$USER"
log ERROR "文件不存在"
```

也可以把调试日志写入标准错误：

```bash
debug() {
    printf 'DEBUG: %s\n' "$*" >&2
}
```

根据环境变量控制是否输出：

```bash
debug() {
    if [[ "${DEBUG:-0}" == "1" ]]; then
        printf 'DEBUG: %s\n' "$*" >&2
    fi
}
```

运行时开启：

```bash
DEBUG=1 ./script.sh
```

---

## 9. 检查变量引用问题

错误示例：

```bash
file="my file.txt"
cat $file
```

Shell 会把它拆成两个参数：

```text
my
file.txt
```

正确写法：

```bash
cat "$file"
```

一般情况下，变量都应加双引号：

```bash
"$name"
"$file"
"$dir"
```

数组则使用：

```bash
files=("a.txt" "my file.txt")

for file in "${files[@]}"; do
    echo "$file"
done
```

---

## 10. 检查脚本实际使用的解释器

查看脚本第一行：

```bash
head -n 1 script.sh
```

例如：

```bash
#!/usr/bin/env bash
```

不要直接这样运行 Bash 脚本：

```bash
sh script.sh
```

因为某些系统中的 `sh` 不是 Bash，可能不支持以下语法：

```bash
[[ ... ]]
(( ... ))
arrays=()
function_name() { ...; }
```

建议：

```bash
bash script.sh
```

或者：

```bash
chmod +x script.sh
./script.sh
```

---

## 11. 使用 ShellCheck

ShellCheck 是非常实用的 Shell 静态检查工具：

```bash
shellcheck script.sh
```

它可以发现：

* 变量没有加引号。
* 未定义变量。
* 无效条件判断。
* 数组使用错误。
* 管道和重定向问题。
* Bash 与 POSIX Shell 的兼容问题。

例如：

```bash
files=$(ls *.txt)
for file in $files; do
    echo $file
done
```

ShellCheck 通常会提示该写法无法正确处理带空格的文件名。

更稳妥的写法：

```bash
for file in ./*.txt; do
    [[ -e "$file" ]] || continue
    printf '%s\n' "$file"
done
```

---

## 12. 调试正在运行的 Shell 脚本

查看进程：

```bash
ps -ef | grep script.sh
```

查看进程树：

```bash
pstree -p
```

查看进程打开的文件：

```bash
lsof -p 进程号
```

查看系统调用：

```bash
strace -f -p 进程号
```

直接跟踪脚本：

```bash
strace -f -o trace.log ./script.sh
```

常见用途：

* 脚本卡住不动。
* 文件找不到。
* 权限不足。
* 网络连接失败。
* 子进程没有正常退出。

---

## 推荐的脚本调试模板

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

trap 'rc=$?; printf "ERROR: file=%s line=%s command=%q exit=%s\n" \
    "${BASH_SOURCE[0]}" "$LINENO" "$BASH_COMMAND" "$rc" >&2' ERR

debug() {
    if [[ "${DEBUG:-0}" == "1" ]]; then
        printf 'DEBUG: %s\n' "$*" >&2
    fi
}

main() {
    debug "脚本开始"

    local input="${1:-}"

    if [[ -z "$input" ]]; then
        printf '用法：%s <文件名>\n' "$0" >&2
        return 1
    fi

    if [[ ! -f "$input" ]]; then
        printf '文件不存在：%s\n' "$input" >&2
        return 1
    fi

    debug "处理文件：$input"
    wc -l -- "$input"
}

main "$@"
```

普通运行：

```bash
./script.sh test.txt
```

开启调试日志：

```bash
DEBUG=1 ./script.sh test.txt
```

显示完整执行过程：

```bash
bash -x ./script.sh test.txt
```

实际排查时，通常按这个顺序进行：

```bash
bash -n script.sh
shellcheck script.sh
bash -x script.sh
```

这三步可以解决大多数 Shell 脚本问题。