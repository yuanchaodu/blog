---
title: Linux 循环结构
section: IT
category: Linux
---

# Linux 循环结构

<img src="images/Linux.svg" width="300">

## Linux 中的循环结构

Linux 本身是操作系统，通常所说的“Linux 循环结构”，主要是指 **Shell 脚本中的循环语句**。

循环结构的作用是：**重复执行一段命令，直到满足结束条件。**

例如，需要创建 10 个文件，不必手工执行 10 次 `touch`，可以用循环一次完成。

Shell 中常用的循环结构有三种：

1. `for` 循环
2. `while` 循环
3. `until` 循环

---

## 一、for 循环

`for` 循环适合处理一组已经知道的内容，例如文件列表、数字范围或用户列表。

### 基本格式

```bash
for 变量 in 数据列表
do
    执行命令
done
```

### 示例：依次输出多个名称

```bash
#!/bin/bash

for name in Linux Windows macOS
do
    echo "当前系统：$name"
done
```

运行结果：

```text
当前系统：Linux
当前系统：Windows
当前系统：macOS
```

这里的执行过程是：

* 第一次，`name` 的值是 `Linux`
* 第二次，`name` 的值是 `Windows`
* 第三次，`name` 的值是 `macOS`

可以把它理解为：从篮子里依次取出每个物品，并对每个物品执行相同操作。

---

### 示例：数字循环

```bash
for i in {1..5}
do
    echo "第 $i 次执行"
done
```

输出：

```text
第 1 次执行
第 2 次执行
第 3 次执行
第 4 次执行
第 5 次执行
```

### 指定步长

```bash
for i in {1..10..2}
do
    echo "$i"
done
```

输出：

```text
1
3
5
7
9
```

其中：

* `1` 是开始值
* `10` 是结束值
* `2` 是每次增加的数值

---

### C 语言风格的 for 循环

```bash
for ((i=1; i<=5; i++))
do
    echo "当前数字：$i"
done
```

其中：

```bash
i=1
```

表示初始值。

```bash
i<=5
```

表示继续循环的条件。

```bash
i++
```

表示每次循环后，`i` 增加 1。

---

## 二、while 循环

`while` 循环表示：

> 当条件成立时，持续执行循环。

### 基本格式

```bash
while 条件
do
    执行命令
done
```

### 示例：输出 1 到 5

```bash
#!/bin/bash

i=1

while [ $i -le 5 ]
do
    echo "当前数字：$i"
    i=$((i + 1))
done
```

这里：

```bash
[ $i -le 5 ]
```

表示判断 `i` 是否小于或等于 5。

```bash
i=$((i + 1))
```

表示每次循环后，让 `i` 增加 1。

注意：如果忘记修改 `i`，条件会一直成立，脚本可能进入死循环。

---

### while 读取文件

假设 `users.txt` 内容如下：

```text
zhangsan
lisi
wangwu
```

可以逐行读取：

```bash
while read -r line
do
    echo "当前用户：$line"
done < users.txt
```

这种写法常用于：

* 读取配置文件
* 处理日志
* 批量处理用户或设备名单
* 读取服务器地址列表

---

## 三、until 循环

`until` 和 `while` 正好相反。

`until` 表示：

> 当条件不成立时，持续执行；条件成立后结束。

### 基本格式

```bash
until 条件
do
    执行命令
done
```

### 示例：输出 1 到 5

```bash
#!/bin/bash

i=1

until [ $i -gt 5 ]
do
    echo "当前数字：$i"
    i=$((i + 1))
done
```

这里的条件是：

```bash
[ $i -gt 5 ]
```

意思是判断 `i` 是否大于 5。

当 `i` 还没有大于 5 时，循环继续；当 `i` 变成 6 时，循环结束。

---

## 四、循环控制语句

循环中常用两个控制命令：

* `break`：立即结束整个循环
* `continue`：跳过本次循环，进入下一次循环

---

### break 示例

```bash
for i in {1..10}
do
    if [ $i -eq 5 ]
    then
        break
    fi

    echo "$i"
done
```

输出：

```text
1
2
3
4
```

当 `i` 等于 5 时，`break` 直接结束整个循环。

---

### continue 示例

```bash
for i in {1..5}
do
    if [ $i -eq 3 ]
    then
        continue
    fi

    echo "$i"
done
```

输出：

```text
1
2
4
5
```

当 `i` 等于 3 时，本次循环被跳过，但后面的循环仍然继续。

---

## 五、无限循环

有时需要程序一直运行，例如持续监控服务状态。

### 写法一

```bash
while true
do
    echo "程序正在运行"
    sleep 5
done
```

### 写法二

```bash
while :
do
    echo "程序正在运行"
    sleep 5
done
```

`sleep 5` 表示暂停 5 秒，否则循环会高速执行，占用较多 CPU。

可以使用：

```text
Ctrl + C
```

终止循环。

---

## 六、嵌套循环

一个循环内部还可以再放一个循环，这叫嵌套循环。

```bash
for i in {1..3}
do
    for j in {1..2}
    do
        echo "i=$i，j=$j"
    done
done
```

输出：

```text
i=1，j=1
i=1，j=2
i=2，j=1
i=2，j=2
i=3，j=1
i=3，j=2
```

外层循环每执行一次，内层循环会完整执行一遍。

---

## 七、常见判断运算符

### 数字比较

| 运算符   | 含义    |
| ----- | ----- |
| `-eq` | 等于    |
| `-ne` | 不等于   |
| `-gt` | 大于    |
| `-ge` | 大于或等于 |
| `-lt` | 小于    |
| `-le` | 小于或等于 |

示例：

```bash
[ $i -le 10 ]
```

表示 `i` 小于或等于 10。

### 字符串比较

```bash
[ "$name" = "admin" ]
```

表示判断变量 `name` 是否等于 `admin`。

变量最好加双引号，避免变量为空时出现判断错误。

---

## 八、实际应用示例

### 批量创建文件

```bash
for i in {1..5}
do
    touch "file${i}.txt"
done
```

会创建：

```text
file1.txt
file2.txt
file3.txt
file4.txt
file5.txt
```

---

### 批量检查服务器连通性

```bash
for ip in 192.168.1.10 192.168.1.11 192.168.1.12
do
    if ping -c 1 -W 1 "$ip" > /dev/null 2>&1
    then
        echo "$ip 可以访问"
    else
        echo "$ip 无法访问"
    fi
done
```

其中：

* `-c 1`：只发送一次测试
* `-W 1`：等待 1 秒
* `> /dev/null 2>&1`：隐藏命令输出
* `$?` 或直接使用 `if ping`：判断命令是否执行成功

---

### 定时检查服务状态

```bash
while true
do
    if systemctl is-active --quiet nginx
    then
        echo "$(date) nginx 服务正常"
    else
        echo "$(date) nginx 服务异常"
    fi

    sleep 60
done
```

这个脚本每隔 60 秒检查一次 Nginx 服务。

---

## 九、三种循环的区别

| 循环类型    | 适用情况             |
| ------- | ---------------- |
| `for`   | 已知循环次数，或需要遍历一组数据 |
| `while` | 条件成立时继续执行        |
| `until` | 条件不成立时继续执行       |

简单记忆：

* `for`：逐个处理
* `while`：满足条件就继续
* `until`：直到条件满足才停止

在实际 Shell 脚本中，`for` 和 `while` 使用得最多。