---
title: Linux Shell 数组
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AoGDW
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/177'
---

# Linux Shell 数组

<img src="images/Linux.svg" width="300">

## Linux Shell 数组

Shell 数组可以在一个变量中保存多个值，适合存放文件名、服务器地址、参数列表等数据。

下面主要介绍 Bash 数组。

### 1. 定义数组

```bash
names=("张三" "李四" "王五")
```

也可以逐个赋值：

```bash
names[0]="张三"
names[1]="李四"
names[2]="王五"
```

数组下标通常从 `0` 开始。

### 2. 读取数组元素

读取第一个元素：

```bash
echo "${names[0]}"
```

读取全部元素：

```bash
echo "${names[@]}"
```

推荐使用双引号，避免元素中包含空格时被错误拆分。

```bash
files=("文件一.txt" "重要 文件.txt")

printf '%s\n' "${files[@]}"
```

### 3. 获取数组长度

获取数组元素数量：

```bash
echo "${#names[@]}"
```

输出：

```text
3
```

获取某个元素的字符串长度：

```bash
echo "${#names[0]}"
```

### 4. 遍历数组

按元素遍历：

```bash
for name in "${names[@]}"; do
    echo "姓名：$name"
done
```

按下标遍历：

```bash
for index in "${!names[@]}"; do
    echo "下标：$index，值：${names[$index]}"
done
```

其中：

```bash
"${!names[@]}"
```

表示获取数组中所有已使用的下标。

### 5. 添加元素

在数组末尾添加元素：

```bash
names+=("赵六")
```

一次添加多个元素：

```bash
names+=("孙七" "周八")
```

### 6. 修改元素

```bash
names[1]="李明"
```

修改后，第二个元素由“李四”变成“李明”。

### 7. 删除元素

删除指定元素：

```bash
unset 'names[1]'
```

删除整个数组：

```bash
unset names
```

需要注意，删除中间元素后，其他元素的下标不会自动向前移动。

```bash
names=("张三" "李四" "王五")
unset 'names[1]'

for index in "${!names[@]}"; do
    echo "$index: ${names[$index]}"
done
```

输出类似：

```text
0: 张三
2: 王五
```

### 8. 数组切片

从下标 `1` 开始，获取两个元素：

```bash
echo "${names[@]:1:2}"
```

格式为：

```bash
"${数组名[@]:起始位置:数量}"
```

### 9. 命令结果保存到数组

例如获取当前目录下的所有 `.log` 文件：

```bash
mapfile -t log_files < <(find . -maxdepth 1 -type f -name '*.log')
```

遍历结果：

```bash
for file in "${log_files[@]}"; do
    echo "$file"
done
```

推荐使用 `mapfile`，不要简单写成：

```bash
files=($(find . -type f))
```

后一种写法会按照空格和换行拆分内容，遇到带空格的文件名时容易出错。

### 10. 关联数组

普通数组使用数字作为下标。关联数组可以使用字符串作为键，类似其他语言中的字典或 Map。

定义关联数组：

```bash
declare -A user
```

赋值：

```bash
user[name]="张三"
user[department]="信息中心"
user[role]="信息工程师"
```

读取：

```bash
echo "${user[name]}"
echo "${user[department]}"
```

遍历：

```bash
for key in "${!user[@]}"; do
    echo "$key：${user[$key]}"
done
```

关联数组需要 Bash 4.0 或更高版本。

### 11. `${array[@]}` 和 `${array[*]}` 的区别

在双引号中，两者含义不同。

```bash
"${array[@]}"
```

每个数组元素仍然是一个独立参数。

```bash
"${array[*]}"
```

所有元素会合并成一个字符串。

例如：

```bash
items=("hello world" "Linux Shell")

printf '<%s>\n' "${items[@]}"
```

输出：

```text
<hello world>
<Linux Shell>
```

而：

```bash
printf '<%s>\n' "${items[*]}"
```

输出：

```text
<hello world Linux Shell>
```

因此，遍历数组和传递命令参数时，通常应使用：

```bash
"${array[@]}"
```

### 完整示例

```bash
#!/usr/bin/env bash

servers=(
    "192.168.1.10"
    "192.168.1.11"
    "192.168.1.12"
)

servers+=("192.168.1.13")

echo "服务器数量：${#servers[@]}"

for index in "${!servers[@]}"; do
    echo "服务器 $index：${servers[$index]}"
done
```

核心写法可以记住四个：

```bash
array=("a" "b" "c")       # 定义数组
echo "${array[0]}"         # 读取单个元素
echo "${array[@]}"         # 读取全部元素
echo "${#array[@]}"        # 获取元素数量
```
