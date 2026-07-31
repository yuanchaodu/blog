---
title: Linux Shell 最佳实践
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoKgZ
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/184'
---

<img src="images/Linux.svg" width="300">

# Linux Shell 最佳实践

Linux Shell 是系统管理和自动化的重要工具。它适合处理文件、调用系统命令、部署应用和编排简单任务。但 Shell 语法存在许多隐含规则，脚本“能够运行”并不代表“稳定可靠”。

下面以 Bash 为主介绍实践建议。

## 一、先选择正确的 Shell

脚本开头应明确解释器：

```bash
#!/usr/bin/env bash
```

如果脚本只使用 POSIX 标准语法，可以写：

```sh
#!/bin/sh
```

两者不要混用：

* `sh`：兼容性更好，适合跨 Unix 系统运行。
* `bash`：功能更丰富，支持数组、`[[ ]]`、正则匹配等。
* 使用了 Bash 特性，就不要声明为 `/bin/sh`。

POSIX Shell 定义了标准的命令、变量展开、管道、重定向和流程控制语法，是编写可移植脚本的基础。([Open Group][1])

---

## 二、限制 Shell 脚本的规模

Shell 最适合：

* 系统命令编排；
* 文件批量处理；
* 环境初始化；
* 部署和运维脚本；
* 简单的数据过滤和转换。

Shell 不适合：

* 复杂业务逻辑；
* 大量数据处理；
* 多层对象和数据结构；
* 复杂并发；
* 长期运行的大型程序。

一般来说，脚本超过几百行，或者出现大量嵌套判断时，应考虑改用 Python、Go 等语言。Google Shell 风格指南也建议，Shell 主要用于较小的工具或命令包装程序。([Google GitHub][2])

---

## 三、启用严格模式，但不要盲目依赖

Bash 脚本常使用：

```bash
set -Eeuo pipefail
```

含义如下：

```bash
set -e          # 命令失败时尽快退出
set -u          # 使用未定义变量时报错
set -o pipefail # 管道中任何命令失败，整个管道返回失败
set -E          # ERR trap 可在函数等环境中继承
```

`pipefail` 很重要。默认情况下，管道的退出状态通常由最后一个命令决定：

```bash
generate_data | upload_data
```

即使 `generate_data` 失败，只要 `upload_data` 返回成功，脚本仍可能认为操作成功。启用 `pipefail` 后，可以发现管道前部的错误。([GNU][3])

不过，`set -e` 存在一些容易误解的例外，例如条件判断、逻辑运算和部分子 Shell 场景。关键操作仍应显式处理：

```bash
if ! cp "$source_file" "$target_file"; then
  printf '复制文件失败：%s\n' "$source_file" >&2
  exit 1
fi
```

不要把严格模式当成完整的异常处理机制。

---

## 四、变量几乎总要加双引号

推荐：

```bash
rm -- "$file"
cp -- "$source" "$target"
printf '%s\n' "$username"
```

不推荐：

```bash
rm $file
cp $source $target
echo $username
```

假设变量值为：

```text
/data/Monthly Report.txt
```

未加引号时，Shell 会把它拆成两个参数。变量中若包含通配符，还可能发生路径扩展。

常见规则是：

```bash
"$variable"
"${array[@]}"
"$(command)"
```

有意进行单词拆分或通配符展开时，才省略引号。

---

## 五、变量引用使用花括号

推荐：

```bash
backup_file="${filename}.bak"
log_file="${log_dir}/application.log"
```

不推荐：

```bash
backup_file="$filename.bak"
```

简单情况下两者结果相同，但花括号能够明确变量边界，可读性更好。

---

## 六、使用 `printf`，少用 `echo`

推荐：

```bash
printf '处理文件：%s\n' "$file"
printf '错误：%s\n' "$message" >&2
```

`echo` 对 `-n`、反斜杠等内容的处理在不同环境下可能存在差异。`printf` 的输出格式更明确，也更适合编写可移植脚本。

---

## 七、命令替换使用 `$()`

推荐：

```bash
current_date="$(date '+%F')"
file_count="$(find "$data_dir" -type f | wc -l)"
```

不推荐旧式反引号：

```bash
current_date=`date '+%F'`
```

`$()` 更容易阅读，也支持嵌套：

```bash
result="$(process_file "$(find_latest_file)")"
```

---

## 八、条件判断优先使用 `[[ ]]`

Bash 脚本中推荐：

```bash
if [[ -f "$config_file" ]]; then
  printf '配置文件存在\n'
fi
```

字符串比较：

```bash
if [[ "$environment" == "production" ]]; then
  deploy
fi
```

正则匹配：

```bash
if [[ "$version" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  printf '版本号格式正确\n'
fi
```

`[[ ]]` 比 `[ ]` 更不容易受到单词拆分和通配符展开的影响。

但 `[[ ]]` 是 Bash 语法。如果脚本声明为 `/bin/sh`，应使用 POSIX 的 `[ ]`：

```sh
if [ -f "$config_file" ]; then
  printf '%s\n' '配置文件存在'
fi
```

---

## 九、命令参数前使用 `--`

涉及文件名的命令建议写成：

```bash
rm -- "$file"
mv -- "$source" "$target"
grep -- "$keyword" "$file"
```

假设文件名是：

```text
-rf
```

如果没有 `--`，命令可能把文件名误认为参数。`--` 表示后面的内容全部按普通参数处理。

需要注意，并不是所有 Unix 命令都支持 `--`。编写跨平台脚本时应查看对应命令规范。

---

## 十、正确处理文件名

不要这样遍历文件：

```bash
for file in $(find "$directory" -type f); do
  process "$file"
done
```

文件名中出现空格、换行符或通配符时，这种写法会出错。

推荐使用空字符分隔：

```bash
while IFS= read -r -d '' file; do
  process_file "$file"
done < <(find "$directory" -type f -print0)
```

或者在简单情况下直接使用通配符：

```bash
shopt -s nullglob

for file in "$directory"/*.csv; do
  process_file "$file"
done
```

`nullglob` 可以避免没有匹配文件时，把字面量 `*.csv` 当成文件名处理。

---

## 十一、读取文本时保留原始内容

推荐：

```bash
while IFS= read -r line; do
  printf '%s\n' "$line"
done < "$input_file"
```

其中：

* `IFS=`：避免删除行首、行尾空白；
* `-r`：避免把反斜杠当作转义符；
* 重定向放在循环末尾，避免无意义的管道子 Shell。

不推荐：

```bash
cat "$input_file" | while read line; do
  echo "$line"
done
```

---

## 十二、使用函数拆分逻辑

推荐：

```bash
log_info() {
  printf '[INFO] %s\n' "$*"
}

validate_file() {
  local file="$1"

  if [[ ! -f "$file" ]]; then
    printf '文件不存在：%s\n' "$file" >&2
    return 1
  fi
}

main() {
  local input_file="${1:-}"

  validate_file "$input_file"
  log_info "开始处理 $input_file"
}

main "$@"
```

函数应尽量做到：

* 一个函数只完成一类工作；
* 使用参数传递数据；
* 使用返回码表示成功或失败；
* 少依赖全局变量；
* 函数名使用小写字母和下划线。

Google Shell 风格指南建议函数名使用小写字母，并用下划线分隔单词。([Google GitHub][2])

---

## 十三、函数内部变量使用 `local`

推荐：

```bash
create_backup() {
  local source_file="$1"
  local backup_file="${source_file}.bak"

  cp -- "$source_file" "$backup_file"
}
```

否则，变量默认是全局变量，可能意外覆盖脚本其他位置的同名变量。

常量可以使用只读变量：

```bash
readonly APP_NAME='data-backup'
readonly DEFAULT_TIMEOUT=30
```

环境变量可以使用大写：

```bash
export HTTP_PROXY='http://proxy.example.com:8080'
```

普通局部变量建议小写：

```bash
local config_file
local retry_count
```

---

## 十四、检查输入参数

不要直接相信位置参数：

```bash
input_file="$1"
```

在 `set -u` 环境下，没有参数时还会直接报错。

推荐：

```bash
usage() {
  printf '用法：%s <输入文件>\n' "${0##*/}" >&2
}

main() {
  if (( $# != 1 )); then
    usage
    return 2
  fi

  local input_file="$1"

  if [[ ! -r "$input_file" ]]; then
    printf '文件不存在或不可读：%s\n' "$input_file" >&2
    return 1
  fi
}
```

退出码可以约定为：

* `0`：成功；
* `1`：一般运行错误；
* `2`：参数使用错误；
* 其他值：根据项目统一定义。

---

## 十五、使用 `getopts` 解析选项

对于带参数的脚本，使用 `getopts`：

```bash
usage() {
  printf '用法：%s [-v] [-o 输出目录] 输入文件\n' "${0##*/}"
}

verbose=false
output_dir='.'

while getopts ':vo:h' option; do
  case "$option" in
    v)
      verbose=true
      ;;
    o)
      output_dir="$OPTARG"
      ;;
    h)
      usage
      exit 0
      ;;
    :)
      printf '选项 -%s 缺少参数\n' "$OPTARG" >&2
      exit 2
      ;;
    \?)
      printf '未知选项：-%s\n' "$OPTARG" >&2
      exit 2
      ;;
  esac
done

shift $((OPTIND - 1))
```

这比手工判断 `$1`、`$2` 更稳定。

---

## 十六、临时文件使用 `mktemp`

不要使用固定临时文件名：

```bash
temp_file="/tmp/app.tmp"
```

它可能引发文件覆盖、并发冲突或符号链接安全问题。

推荐：

```bash
temp_dir="$(mktemp -d)"

cleanup() {
  rm -rf -- "$temp_dir"
}

trap cleanup EXIT
```

创建后自动清理：

```bash
trap cleanup EXIT INT TERM
```

不要在清理命令中使用未经检查的变量：

```bash
rm -rf "$temp_dir"
```

应先保证变量已赋值且目录确实由当前脚本创建。

---

## 十七、危险操作前进行保护

涉及删除、覆盖和系统修改时，应先检查目标：

```bash
delete_directory() {
  local target="$1"

  if [[ -z "$target" || "$target" == "/" ]]; then
    printf '拒绝删除危险路径：%q\n' "$target" >&2
    return 1
  fi

  if [[ ! -d "$target" ]]; then
    printf '目录不存在：%s\n' "$target" >&2
    return 1
  fi

  rm -rf -- "$target"
}
```

重要原则：

* 路径变量不能为空；
* 不允许意外操作根目录；
* 先判断目标类型；
* 尽量使用绝对路径；
* 不必要时不要以 root 身份运行；
* 重要任务可以先提供 `--dry-run` 模式。

---

## 十八、不要使用 `eval`

危险写法：

```bash
eval "$user_input"
```

`eval` 会把字符串重新当作 Shell 代码执行。只要内容来自参数、文件、网络或其他外部来源，就可能产生命令注入风险。

需要组织命令参数时，应使用数组：

```bash
command_args=(
  rsync
  -a
  --delete
  "$source_dir/"
  "$target_dir/"
)

"${command_args[@]}"
```

不要拼接命令字符串：

```bash
command="rsync -a $source_dir $target_dir"
$command
```

数组能够保持参数边界，尤其适合包含空格的路径。

---

## 十九、日志输出到标准错误

普通结果输出到标准输出：

```bash
printf '%s\n' "$result"
```

日志和错误输出到标准错误：

```bash
printf '[INFO] 开始执行\n' >&2
printf '[ERROR] 操作失败\n' >&2
```

可以统一封装：

```bash
log() {
  local level="$1"
  shift

  printf '%s [%s] %s\n' \
    "$(date '+%F %T')" \
    "$level" \
    "$*" >&2
}
```

这样可以将程序结果和运行日志分开：

```bash
result="$(./script.sh 2>script.log)"
```

日志中不要打印密码、令牌、密钥和完整连接字符串。

---

## 二十、谨慎使用调试模式

调试脚本可以使用：

```bash
bash -x script.sh
```

或者：

```bash
set -x
some_commands
set +x
```

但 `set -x` 会打印展开后的命令，可能泄露密码和令牌：

```bash
set +x
login --password "$password"
set -x
```

生产脚本建议通过环境变量控制调试：

```bash
if [[ "${DEBUG:-false}" == "true" ]]; then
  set -x
fi
```

---

## 二十一、检查外部命令是否存在

脚本依赖外部工具时，应提前检查：

```bash
require_command() {
  local command_name="$1"

  if ! command -v "$command_name" >/dev/null 2>&1; then
    printf '缺少必要命令：%s\n' "$command_name" >&2
    return 1
  fi
}

require_command curl
require_command jq
```

使用：

```bash
command -v jq
```

比使用 `which jq` 更适合写入脚本，因为 `command -v` 是 Shell 内建方式。

---

## 二十二、固定不可控的运行环境

脚本可能受环境变量影响，例如：

* `PATH`；
* `IFS`；
* `LANG`、`LC_ALL`；
* 当前工作目录；
* 用户的别名和函数。

关键脚本可显式设置：

```bash
export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
export LC_ALL=C
```

确定脚本自身目录：

```bash
script_dir="$(
  cd -- "$(dirname -- "${BASH_SOURCE[0]}")" >/dev/null 2>&1
  pwd -P
)"
```

不要假设脚本总是在脚本文件所在目录运行。

---

## 二十三、避免解析面向人的命令输出

不推荐：

```bash
disk_usage="$(df -h | grep '/data' | awk '{print $5}')"
```

`df -h` 主要供人阅读，输出可能受到语言、平台和格式变化影响。

更好的做法是：

* 优先使用命令的机器可读选项；
* 使用稳定的输出格式；
* 必要时设置 `LC_ALL=C`；
* 优先使用 JSON、空字符分隔或明确字段格式；
* 避免解析 `ls` 输出。

例如遍历文件不要使用：

```bash
for file in $(ls "$directory"); do
  ...
done
```

直接使用：

```bash
for file in "$directory"/*; do
  ...
done
```

---

## 二十四、使用静态检查工具

### 1. 语法检查

```bash
bash -n script.sh
```

它只检查语法，不执行脚本。

### 2. ShellCheck

```bash
shellcheck script.sh
```

ShellCheck 可以发现：

* 变量未加引号；
* 无效条件判断；
* 数组使用错误；
* 命令替换问题；
* 不可靠的文件遍历；
* 变量作用域问题；
* 部分兼容性问题。

ShellCheck 官方将其定位为 Shell 脚本静态分析工具，用于发现常见语法错误和容易产生异常行为的语义问题。([GitHub][4])

不要随意屏蔽警告。确实需要屏蔽时，应写明原因：

```bash
# 这里需要有意进行单词拆分，内容由程序内部生成。
# shellcheck disable=SC2086
command $generated_options
```

### 3. 格式化

可使用 `shfmt` 统一缩进和格式：

```bash
shfmt -w script.sh
```

建议在持续集成中执行：

```bash
bash -n script.sh
shellcheck script.sh
shfmt -d script.sh
```

---

## 二十五、一个推荐的 Bash 脚本模板

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

readonly SCRIPT_NAME="${0##*/}"

log_info() {
  printf '%s [INFO] %s\n' "$(date '+%F %T')" "$*" >&2
}

log_error() {
  printf '%s [ERROR] %s\n' "$(date '+%F %T')" "$*" >&2
}

usage() {
  printf '用法：%s <输入文件> <输出目录>\n' "$SCRIPT_NAME"
}

require_command() {
  local command_name="$1"

  if ! command -v "$command_name" >/dev/null 2>&1; then
    log_error "缺少必要命令：$command_name"
    return 1
  fi
}

main() {
  if (( $# != 2 )); then
    usage >&2
    return 2
  fi

  local input_file="$1"
  local output_dir="$2"

  if [[ ! -r "$input_file" ]]; then
    log_error "输入文件不存在或不可读：$input_file"
    return 1
  fi

  if [[ ! -d "$output_dir" ]]; then
    log_error "输出目录不存在：$output_dir"
    return 1
  fi

  require_command awk

  log_info "开始处理文件：$input_file"

  awk '{ print }' "$input_file" \
    >"${output_dir}/result.txt"

  log_info "处理完成"
}

main "$@"
```

---

## 核心检查清单

提交或上线 Shell 脚本前，至少检查以下内容：

1. 解释器是否与语法一致；
2. 变量是否正确加双引号；
3. 输入参数是否经过验证；
4. 文件名是否支持空格等特殊字符；
5. 是否存在危险的 `rm -rf`；
6. 是否使用了 `eval` 或命令字符串拼接；
7. 临时文件是否通过 `mktemp` 创建并清理；
8. 密码和令牌是否会进入日志；
9. 外部命令依赖是否经过检查；
10. 是否通过 `bash -n` 和 ShellCheck；
11. 是否在普通用户权限下测试；
12. 是否测试失败、空输入、重复运行和并发运行场景。

一句话概括：**把 Shell 脚本当作正式程序，而不是临时命令的简单堆叠。**这样写出的脚本才更容易维护，也更不容易在生产环境中“踩雷”。

[1]: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/V3_chap02.html "2. Shell Command Language"
[2]: https://google.github.io/styleguide/shellguide.html "Shell Style Guide"
[3]: https://www.gnu.org/s/bash/manual/bash.html "Bash Reference Manual"
[4]: https://github.com/koalaman/shellcheck "ShellCheck, a static analysis tool for shell scripts"
