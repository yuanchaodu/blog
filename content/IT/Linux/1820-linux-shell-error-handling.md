---
title: Linux shell 异常处理
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoHK7
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/182'
---

# Linux shell 异常处理

<img src="images/Linux.svg" width="300">

在 Linux Shell 脚本中，“异常处理”通常不是捕获异常对象，而是通过**退出状态码、条件判断和 `trap` 信号捕获**来发现并处理错误。

## 1. 判断命令是否执行成功

Linux 命令执行后都会返回一个状态码：

* `0`：执行成功
* 非 `0`：执行失败

```bash
#!/bin/bash

cp source.txt /backup/

if [ $? -eq 0 ]; then
    echo "文件复制成功"
else
    echo "文件复制失败"
    exit 1
fi
```

更推荐直接把命令写进 `if`，避免 `$?` 被后续命令覆盖：

```bash
if cp source.txt /backup/; then
    echo "文件复制成功"
else
    echo "文件复制失败"
    exit 1
fi
```

## 2. 开启严格模式

脚本开头通常可以加入：

```bash
set -Eeuo pipefail
```

各参数的作用如下。

### `set -e`

命令执行失败时，立即结束脚本。

```bash
set -e

mkdir /root/test
echo "如果 mkdir 失败，这句话通常不会执行"
```

但在 `if`、`while`、`&&`、`||` 等结构中，`set -e` 的行为会有例外，因此不能完全依赖它。

### `set -u`

使用未定义变量时，立即报错退出。

```bash
set -u

echo "$user_name"
```

如果 `user_name` 未定义，脚本会退出。

对于允许为空的变量，可以写成：

```bash
echo "${user_name:-默认值}"
```

### `set -o pipefail`

管道中的任意命令失败，整个管道都视为失败。

```bash
set -o pipefail

cat not_exist.txt | grep "error"
```

没有 `pipefail` 时，Shell 可能只看最后一个 `grep` 的状态，导致前面的错误被忽略。

### `set -E`

让 `ERR` 捕获器在函数、子 Shell 等环境中继续生效。

---

## 3. 使用 `trap` 捕获错误

`trap` 可以在错误发生时执行统一处理。

```bash
#!/bin/bash

set -Eeuo pipefail

trap 'echo "脚本执行失败，位置：第 ${LINENO} 行，命令：${BASH_COMMAND}" >&2' ERR

cp not_exist.txt /tmp/
```

可能输出：

```text
脚本执行失败，位置：第 7 行，命令：cp not_exist.txt /tmp/
```

常用变量：

* `${LINENO}`：当前行号
* `${BASH_COMMAND}`：当前执行的命令
* `$?`：上一条命令的退出状态码
* `${FUNCNAME[0]}`：当前函数名称

更完整的写法：

```bash
#!/bin/bash

set -Eeuo pipefail

error_handler() {
    local exit_code=$?
    local line_no=$1

    echo "错误：脚本执行失败" >&2
    echo "行号：${line_no}" >&2
    echo "命令：${BASH_COMMAND}" >&2
    echo "退出码：${exit_code}" >&2

    exit "${exit_code}"
}

trap 'error_handler ${LINENO}' ERR
```

## 4. 脚本退出时清理临时资源

可以使用 `EXIT` 捕获器删除临时文件、释放锁或恢复环境。

```bash
#!/bin/bash

set -Eeuo pipefail

temp_file=$(mktemp)

cleanup() {
    rm -f "$temp_file"
    echo "临时文件已清理"
}

trap cleanup EXIT

echo "处理中……" > "$temp_file"
```

无论脚本正常结束还是异常退出，`cleanup` 一般都会执行。

## 5. 同时处理错误和清理工作

实际脚本中，可以分别设置错误处理和退出清理。

```bash
#!/bin/bash

set -Eeuo pipefail

temp_dir=$(mktemp -d)

cleanup() {
    rm -rf "$temp_dir"
}

error_handler() {
    local exit_code=$?
    local line_no=$1

    echo "脚本执行失败：" >&2
    echo "  行号：${line_no}" >&2
    echo "  命令：${BASH_COMMAND}" >&2
    echo "  状态码：${exit_code}" >&2

    exit "$exit_code"
}

trap cleanup EXIT
trap 'error_handler ${LINENO}' ERR

cp source.txt "$temp_dir/"
grep "关键字" "$temp_dir/source.txt"
```

## 6. 主动抛出错误

Shell 没有 `throw`，通常通过输出错误信息并返回非零状态码表示失败。

```bash
check_file() {
    local file=$1

    if [[ ! -f "$file" ]]; then
        echo "错误：文件不存在：$file" >&2
        return 1
    fi
}
```

调用时：

```bash
if ! check_file "/data/config.ini"; then
    echo "配置检查失败"
    exit 1
fi
```

其中：

* `return`：结束函数
* `exit`：结束整个脚本

不要在普通函数中随意使用 `exit`，否则调用函数的主脚本也会直接退出。

## 7. 对可预期错误进行单独处理

有些失败属于正常业务情况，不应交给统一异常处理。

例如，检查进程是否存在：

```bash
if pgrep -x "nginx" >/dev/null; then
    echo "nginx 正在运行"
else
    echo "nginx 未运行"
fi
```

再如，目录不存在时创建：

```bash
backup_dir="/data/backup"

if [[ ! -d "$backup_dir" ]]; then
    mkdir -p "$backup_dir" || {
        echo "错误：无法创建目录：$backup_dir" >&2
        exit 1
    }
fi
```

这里的 `||` 可以理解为：“前面的命令失败，就执行后面的处理”。

## 8. 推荐的通用脚本模板

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

readonly SCRIPT_NAME=$(basename "$0")

log_info() {
    printf '[INFO] %s\n' "$*"
}

log_error() {
    printf '[ERROR] %s\n' "$*" >&2
}

cleanup() {
    log_info "开始清理临时资源"
}

error_handler() {
    local exit_code=$?
    local line_no=$1
    local command=$2

    log_error "脚本执行异常"
    log_error "脚本：${SCRIPT_NAME}"
    log_error "行号：${line_no}"
    log_error "命令：${command}"
    log_error "退出码：${exit_code}"

    exit "$exit_code"
}

trap cleanup EXIT
trap 'error_handler "${LINENO}" "${BASH_COMMAND}"' ERR
trap 'log_error "脚本被中断"; exit 130' INT
trap 'log_error "脚本收到终止信号"; exit 143' TERM

main() {
    log_info "脚本开始执行"

    local source_file="${1:-}"

    if [[ -z "$source_file" ]]; then
        log_error "请指定源文件"
        return 2
    fi

    if [[ ! -f "$source_file" ]]; then
        log_error "文件不存在：${source_file}"
        return 3
    fi

    cp "$source_file" /tmp/

    log_info "脚本执行完成"
}

main "$@"
```

## 9. 实际使用建议

建议遵循以下原则：

1. 脚本开头使用 `set -Eeuo pipefail`。
2. 使用 `trap` 统一记录错误行号、命令和退出码。
3. 对文件不存在、参数缺失等可预期问题，使用 `if` 主动判断。
4. 错误信息输出到标准错误流：

```bash
echo "错误信息" >&2
```

5. 变量始终加双引号：

```bash
rm -f "$file"
```

不要写成：

```bash
rm -f $file
```

6. 清理操作放入 `trap cleanup EXIT`。
7. 不要只依赖 `set -e`，关键命令仍应显式判断结果。

可以简单理解为：`set` 是“总保险”，`trap` 是“事故记录器”，而 `if` 判断是“重点位置的现场检查”。三者配合，Shell 脚本才会更可靠。
