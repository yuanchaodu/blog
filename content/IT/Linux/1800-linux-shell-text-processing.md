---
title: Linux shell 文本处理技巧
section: IT
category: Linux
---

# Linux shell 文本处理技巧

<img src="images/Linux.svg" width="300">

Linux 文本处理的核心思想是：**每个命令只完成一件事，再通过管道把它们连接起来。**

例如：

```bash
grep "ERROR" app.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

这条命令依次完成：查找错误、提取字段、排序、统计、按数量倒序排列。Linux 中很多文本工具既能读取文件，也能接收标准输入，因此特别适合用管道组合。([GNU][1])

---

## 一、准备一份示例数据

下面的示例假设存在文件 `users.txt`：

```text
1001,zhangsan,IT,8500
1002,lisi,Production,7200
1003,wangwu,IT,9200
1004,zhaoliu,Quality,7800
1005,sunqi,Production,7200
```

字段分别表示：

```text
编号,姓名,部门,工资
```

---

## 二、查看文件内容

### 1. `cat`：输出整个文件

```bash
cat users.txt
```

显示行号：

```bash
cat -n users.txt
```

不建议用 `cat` 查看特别大的文件，因为内容会快速滚动。

### 2. `less`：分页查看大文件

```bash
less users.txt
```

常用操作：

```text
空格        向下翻页
b           向上翻页
/ERROR      向下搜索 ERROR
n           查找下一个
N           查找上一个
q           退出
```

查看日志时，`less` 通常比 `cat` 更合适：

```bash
less /var/log/messages
```

### 3. `head` 和 `tail`

查看前 10 行：

```bash
head users.txt
```

查看前 3 行：

```bash
head -n 3 users.txt
```

查看最后 20 行：

```bash
tail -n 20 app.log
```

持续跟踪日志：

```bash
tail -f app.log
```

同时显示最后 100 行并持续跟踪：

```bash
tail -n 100 -f app.log
```

---

## 三、使用 `grep` 查找文本

`grep` 用来按照关键字或正则表达式筛选文本。GNU `grep` 支持基础正则、扩展正则和 Perl 风格正则等匹配方式。([man7.org][2])

### 1. 基本查找

```bash
grep "IT" users.txt
```

输出：

```text
1001,zhangsan,IT,8500
1003,wangwu,IT,9200
```

### 2. 忽略大小写

```bash
grep -i "error" app.log
```

### 3. 显示行号

```bash
grep -n "ERROR" app.log
```

### 4. 反向筛选

显示不包含 `INFO` 的行：

```bash
grep -v "INFO" app.log
```

### 5. 同时匹配多个条件

```bash
grep -E "ERROR|WARN" app.log
```

也可以：

```bash
grep -e "ERROR" -e "WARN" app.log
```

### 6. 只显示匹配内容

```bash
grep -oE '[0-9]{1,3}(\.[0-9]{1,3}){3}' access.log
```

适合从日志中提取 IP 地址。

### 7. 递归搜索目录

```bash
grep -R "database.password" /opt/app/config/
```

显示文件名和行号：

```bash
grep -Rn "database.password" /opt/app/config/
```

排除某类文件：

```bash
grep -Rn "TODO" . --exclude="*.log"
```

排除目录：

```bash
grep -Rn "TODO" . --exclude-dir=".git"
```

### 8. 显示上下文

显示匹配行及其前后 2 行：

```bash
grep -C 2 "ERROR" app.log
```

只显示前 3 行：

```bash
grep -B 3 "ERROR" app.log
```

只显示后 3 行：

```bash
grep -A 3 "ERROR" app.log
```

---

## 四、使用 `cut` 提取字段

`cut` 适合处理字段结构比较固定的文本。

### 1. 按分隔符提取字段

提取姓名：

```bash
cut -d ',' -f 2 users.txt
```

提取姓名和部门：

```bash
cut -d ',' -f 2,3 users.txt
```

提取第 2 到第 4 个字段：

```bash
cut -d ',' -f 2-4 users.txt
```

### 2. 按字符位置提取

```bash
cut -c 1-10 app.log
```

这会提取每行的第 1 到第 10 个字符。

### 注意

`cut` 不适合处理连续空格数量不固定的数据。例如：

```text
zhangsan    IT       8500
lisi        Production 7200
```

这种数据使用 `awk` 更方便。

---

## 五、使用 `sort` 排序

### 1. 普通排序

```bash
sort users.txt
```

### 2. 数字排序

按照第 4 个字段，即工资排序：

```bash
sort -t ',' -k 4,4n users.txt
```

参数含义：

```text
-t ','    使用逗号作为分隔符
-k 4,4    只使用第 4 个字段
-n        按数字排序
```

工资倒序：

```bash
sort -t ',' -k 4,4nr users.txt
```

### 3. 多字段排序

先按部门排序，再按工资倒序：

```bash
sort -t ',' -k 3,3 -k 4,4nr users.txt
```

### 4. 排序并去重

```bash
sort -u users.txt
```

需要注意，带有复杂排序条件时，`sort -u` 和 `sort | uniq` 的判断方式可能并不完全相同。([GNU][3])

### 5. 指定稳定语言环境

脚本中为了获得稳定、可预测的排序结果，可以写：

```bash
LC_ALL=C sort users.txt
```

不同语言环境可能采用不同的字符排序规则。

---

## 六、使用 `uniq` 去重和统计

`uniq` 只能合并**相邻的重复行**，所以通常需要先执行 `sort`。

### 1. 去重

```bash
sort departments.txt | uniq
```

### 2. 统计出现次数

```bash
sort departments.txt | uniq -c
```

### 3. 按出现次数倒序

```bash
sort departments.txt | uniq -c | sort -nr
```

统计用户所属部门：

```bash
cut -d ',' -f 3 users.txt | sort | uniq -c | sort -nr
```

输出类似：

```text
2 Production
2 IT
1 Quality
```

### 4. 只显示重复项

```bash
sort data.txt | uniq -d
```

### 5. 只显示未重复项

```bash
sort data.txt | uniq -u
```

---

## 七、使用 `tr` 替换或删除字符

### 1. 大写转换为小写

```bash
echo "Linux SHELL" | tr 'A-Z' 'a-z'
```

推荐使用字符类：

```bash
echo "Linux SHELL" | tr '[:upper:]' '[:lower:]'
```

### 2. 将多个空格压缩为一个

```bash
tr -s ' ' < data.txt
```

也可以压缩所有空白字符：

```bash
tr -s '[:space:]' ' ' < data.txt
```

### 3. 删除回车符

Windows 文本使用 `CRLF` 换行，Linux 通常使用 `LF`。删除行尾回车：

```bash
tr -d '\r' < windows.txt > linux.txt
```

### 4. 将分隔符转换为换行

```bash
echo "apple,orange,banana" | tr ',' '\n'
```

输出：

```text
apple
orange
banana
```

---

## 八、使用 `sed` 修改文本

`sed` 是流编辑器，可以对输入内容进行替换、删除、选择和转换。它通常逐行处理文本，因此很适合放在管道中。([man7.org][4])

### 1. 替换文本

```bash
sed 's/Production/生产部门/' users.txt
```

默认只替换每行第一次出现的内容。

替换每行所有匹配内容：

```bash
sed 's/ERROR/WARN/g' app.log
```

### 2. 直接修改文件

```bash
sed -i 's/old.example.com/new.example.com/g' config.ini
```

建议修改前生成备份：

```bash
sed -i.bak 's/old.example.com/new.example.com/g' config.ini
```

原文件备份为：

```text
config.ini.bak
```

### 3. 删除行

删除空行：

```bash
sed '/^[[:space:]]*$/d' data.txt
```

删除注释行：

```bash
sed '/^[[:space:]]*#/d' config.ini
```

同时删除空行和注释行：

```bash
sed -e '/^[[:space:]]*#/d' -e '/^[[:space:]]*$/d' config.ini
```

### 4. 输出指定行

输出第 10 行：

```bash
sed -n '10p' app.log
```

输出第 10 到第 20 行：

```bash
sed -n '10,20p' app.log
```

输出包含 `ERROR` 的行：

```bash
sed -n '/ERROR/p' app.log
```

### 5. 替换配置项

原配置：

```text
timeout=30
```

修改为：

```bash
sed -i 's/^timeout=.*/timeout=60/' app.conf
```

使用 `^` 限定行首，可以避免误修改其他位置出现的 `timeout`。

### 6. 使用其他分隔符

处理路径或网址时，可以不用 `/`：

```bash
sed 's#/opt/app#/data/app#g' config.ini
```

这样比下面的写法更清楚：

```bash
sed 's/\/opt\/app/\/data\/app/g' config.ini
```

---

## 九、使用 `awk` 处理结构化文本

`awk` 是专门用于文本数据处理的语言。它按照“匹配条件—执行动作”的方式逐条处理记录，默认把每一行看作一条记录。([man7.org][5])

可以把它理解为一把“小型数据处理瑞士军刀”。

### 1. 输出指定字段

```bash
awk -F ',' '{print $2, $3}' users.txt
```

其中：

```text
-F ','    指定逗号分隔
$1        第一个字段
$2        第二个字段
$NF       最后一个字段
$0        整行内容
```

### 2. 添加输出分隔符

```bash
awk -F ',' 'BEGIN{OFS="\t"} {print $1,$2,$3}' users.txt
```

### 3. 按条件筛选

查找工资大于 8000 的人员：

```bash
awk -F ',' '$4 > 8000 {print $2, $4}' users.txt
```

查找 IT 部门：

```bash
awk -F ',' '$3 == "IT" {print $0}' users.txt
```

多个条件：

```bash
awk -F ',' '$3 == "IT" && $4 > 8000 {print $2, $4}' users.txt
```

### 4. 计算合计和平均值

计算工资总额：

```bash
awk -F ',' '{sum += $4} END {print sum}' users.txt
```

计算平均工资：

```bash
awk -F ',' '{sum += $4} END {if (NR > 0) print sum / NR}' users.txt
```

这里：

```text
NR    当前处理的总行号
END   所有数据处理完成后执行
```

### 5. 按部门汇总

```bash
awk -F ',' '
{
    count[$3]++
    salary[$3] += $4
}
END {
    for (dept in count) {
        print dept, count[dept], salary[dept]
    }
}' users.txt
```

输出内容依次为：

```text
部门 人数 工资总额
```

### 6. 格式化输出

```bash
awk -F ',' '
BEGIN {
    printf "%-12s %-15s %10s\n", "姓名", "部门", "工资"
}
{
    printf "%-12s %-15s %10.2f\n", $2, $3, $4
}' users.txt
```

### 7. 处理日志

假设访问日志的第一个字段是客户端 IP：

```bash
awk '{count[$1]++} END {
    for (ip in count)
        print count[ip], ip
}' access.log | sort -nr | head
```

这可以找出访问次数最多的 IP。

---

## 十、使用 `wc` 统计数量

统计行数：

```bash
wc -l users.txt
```

统计单词数：

```bash
wc -w document.txt
```

统计字节数：

```bash
wc -c document.txt
```

统计字符数：

```bash
wc -m document.txt
```

统计匹配行数：

```bash
grep -c "ERROR" app.log
```

或者：

```bash
grep "ERROR" app.log | wc -l
```

注意：如果一行包含多个 `ERROR`，上面的命令仍然只统计为一行。

统计实际出现次数：

```bash
grep -o "ERROR" app.log | wc -l
```

---

## 十一、文件比较与集合处理

### 1. `diff`：比较两个文件

```bash
diff old.conf new.conf
```

统一格式输出：

```bash
diff -u old.conf new.conf
```

这种格式更容易阅读，也常用于补丁文件。

### 2. `comm`：比较两个已排序文件

```bash
comm file1.txt file2.txt
```

输出分为三列：

```text
只在文件1中
只在文件2中
两个文件都有
```

只显示两个文件共有的内容：

```bash
comm -12 <(sort file1.txt) <(sort file2.txt)
```

只显示文件 1 独有的内容：

```bash
comm -23 <(sort file1.txt) <(sort file2.txt)
```

### 3. 查找两个名单的差异

```bash
grep -Fxv -f old_users.txt new_users.txt
```

含义：

```text
-F    按普通字符串匹配
-x    必须整行一致
-v    显示不匹配行
-f    从文件读取匹配内容
```

这条命令会显示 `new_users.txt` 中新增的用户。

---

## 十二、处理文件名时避免空格问题

下面的写法存在风险：

```bash
for file in $(find . -name "*.log"); do
    echo "$file"
done
```

文件名包含空格、换行或特殊字符时可能出错。

更安全的写法：

```bash
find . -name "*.log" -print0 |
while IFS= read -r -d '' file; do
    echo "$file"
done
```

或者使用 `find -exec`：

```bash
find . -name "*.log" -exec gzip -- {} \;
```

批量传递参数：

```bash
find . -name "*.log" -exec gzip -- {} +
```

`{}` 表示找到的文件，`--` 表示后面的内容都是文件名，避免以 `-` 开头的文件名被误认为选项。

---

## 十三、常见实战组合

### 1. 统计日志级别

```bash
grep -oE 'INFO|WARN|ERROR' app.log |
sort |
uniq -c |
sort -nr
```

### 2. 查看错误最多的日期

假设日志开头格式为：

```text
2026-07-28 10:20:31 ERROR ...
```

命令：

```bash
grep "ERROR" app.log |
awk '{print $1}' |
sort |
uniq -c |
sort -nr
```

### 3. 查找耗时超过 3 秒的请求

假设日志包含：

```text
request=/api/order duration=3500ms
```

命令：

```bash
awk '
match($0, /duration=([0-9]+)ms/, value) {
    if (value[1] > 3000)
        print
}' app.log
```

### 4. 统计访问量最高的 URL

假设 URL 是访问日志的第 7 个字段：

```bash
awk '{print $7}' access.log |
sort |
uniq -c |
sort -nr |
head -20
```

### 5. 提取配置文件有效内容

```bash
sed -e '/^[[:space:]]*#/d' \
    -e '/^[[:space:]]*$/d' \
    app.conf
```

### 6. 批量修改配置文件

```bash
find /opt/app/config -type f -name "*.conf" \
    -exec sed -i.bak \
    's/^timeout=.*/timeout=60/' {} +
```

### 7. 查找重复数据

```bash
sort data.txt | uniq -d
```

显示重复次数：

```bash
sort data.txt | uniq -c | awk '$1 > 1'
```

### 8. 提取两个时间点之间的日志

```bash
awk '
$0 >= "2026-07-28 08:00:00" &&
$0 <= "2026-07-28 09:00:00"
' app.log
```

这种方法要求日志时间位于行首，并且采用可直接按字符排序的固定格式。

---

## 十四、正则表达式常用写法

| 表达式           | 含义         |           |
| ------------- | ---------- | --------- |
| `^ERROR`      | 以 ERROR 开头 |           |
| `ERROR$`      | 以 ERROR 结尾 |           |
| `^$`          | 空行         |           |
| `[0-9]`       | 一个数字       |           |
| `[[:digit:]]` | 一个数字       |           |
| `[[:space:]]` | 空白字符       |           |
| `[^,]+`       | 一个或多个非逗号字符 |           |
| `ERROR\|WARN` | 基础正则中的“或”  |           |
| `ERROR        | WARN`      | 扩展正则中的“或” |
| `[0-9]{4}`    | 连续四个数字     |           |

使用扩展正则：

```bash
grep -E 'ERROR|WARN' app.log
```

匹配日期：

```bash
grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' app.log
```

在 Shell 中，正则表达式一般建议使用单引号：

```bash
grep -E '^[0-9]+$' data.txt
```

这样可以减少 `$`、`*`、`\` 等字符被 Shell 提前解释的问题。

---

## 十五、提高可靠性的技巧

### 1. 先预览，再修改

危险：

```bash
sed -i 's/old/new/g' *.conf
```

建议先执行：

```bash
sed 's/old/new/g' example.conf
```

确认输出正确，再加 `-i`。

### 2. 修改时保留备份

```bash
sed -i.bak 's/old/new/g' *.conf
```

### 3. 变量必须加双引号

不推荐：

```bash
grep ERROR $file
```

推荐：

```bash
grep "ERROR" "$file"
```

否则文件名中的空格和通配符可能引起错误。

### 4. 固定字符串使用 `grep -F`

```bash
grep -F 'a.b[1]' data.txt
```

这里会把 `.`、`[` 和 `]` 当作普通字符，而不是正则符号。

### 5. 命令选项后使用 `--`

```bash
grep -F -- "$keyword" "$file"
```

当关键字或文件名以 `-` 开头时，可以防止它被识别为命令选项。

### 6. 脚本开启严格模式

```bash
set -euo pipefail
```

主要作用：

```text
-e            命令失败时结束脚本
-u            使用未定义变量时报错
-o pipefail   管道中任一命令失败，整个管道视为失败
```

不过 `-e` 存在一些容易误解的行为。正式脚本中仍要对关键命令进行明确的错误判断：

```bash
if ! grep -q "success" result.log; then
    echo "未发现成功标志" >&2
    exit 1
fi
```

---

## 十六、命令选择建议

可以按照下面的思路选择工具：

```text
查找包含某些内容的行       grep
替换、删除或选择某些行     sed
按字段计算和汇总           awk
提取固定字段               cut
字符转换                   tr
排序                       sort
去重和计数                 uniq
统计行数和字符数           wc
比较文件                   diff、comm
查看大文件                 less
实时查看日志               tail -f
```

简单地说：

* **grep 像筛子**：筛出符合条件的行。
* **sed 像流水线编辑器**：边读取边修改。
* **awk 像小型表格处理程序**：按字段筛选、计算和汇总。

掌握下面这组命令，已经可以解决大多数日常文本处理问题：

```bash
grep
sed
awk
cut
sort
uniq
tr
wc
head
tail
less
find
xargs
```

最值得反复练习的组合是：

```bash
grep ... |
awk ... |
sort |
uniq -c |
sort -nr |
head
```

它在日志分析、配置检查、数据核对和系统运维中都很实用。

[1]: https://www.gnu.org/software/coreutils/manual/coreutils.html "GNU Coreutils 9.11"
[2]: https://man7.org/linux/man-pages/man1/grep.1.html "grep(1) - Linux manual page"
[3]: https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html "sort invocation (GNU Coreutils 9.11)"
[4]: https://man7.org/linux/man-pages/man1/sed.1.html "sed(1) - Linux manual page"
[5]: https://man7.org/linux/man-pages/man1/gawk.1.html "gawk(1) - Linux manual page"