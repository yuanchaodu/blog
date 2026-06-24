---
title: Linux 静态库
section: IT
category: Linux
---

# Linux 静态库

<img src="images/Linux.svg" width="300">

Linux **静态库**就是把一组已经编译好的目标文件打包成一个文件，供程序在**编译链接阶段**使用。

常见文件名：

```bash
libxxx.a
```

例如：

```bash
libmylib.a
```

其中：

* `lib` 是固定前缀；
* `mylib` 是库名；
* `.a` 表示静态库。

## 1. 静态库的作用

假设你有很多常用函数，比如日志、字符串处理、计算函数等，不想每个项目都复制源码，就可以把它们编译成静态库。

程序链接静态库后，库中用到的代码会被复制进最终可执行文件。

所以生成的程序通常可以独立运行，不太依赖外部库文件。

## 2. 静态库的制作流程

假设有两个文件：

```c
// add.c
int add(int a, int b) {
    return a + b;
}
```

```c
// add.h
int add(int a, int b);
```

先编译成目标文件：

```bash
gcc -c add.c -o add.o
```

再打包成静态库：

```bash
ar rcs libadd.a add.o
```

查看库内容：

```bash
ar t libadd.a
```

## 3. 使用静态库

主程序：

```c
#include <stdio.h>
#include "add.h"

int main() {
    printf("%d\n", add(2, 3));
    return 0;
}
```

编译链接：

```bash
gcc main.c -L. -ladd -o main
```

含义：

```bash
-L.      # 在当前目录查找库
-ladd    # 链接 libadd.a 或 libadd.so
```

注意：写 `-ladd`，不是写 `-llibadd.a`。

也可以直接写库文件路径：

```bash
gcc main.c ./libadd.a -o main
```

## 4. 静态库和动态库区别

| 对比项     | 静态库     | 动态库           |
| ------- | ------- | ------------- |
| 文件后缀    | `.a`    | `.so`         |
| 链接时间    | 编译时链接   | 运行时加载         |
| 可执行文件大小 | 较大      | 较小            |
| 运行依赖    | 依赖少     | 需要 `.so` 文件存在 |
| 更新方式    | 需重新编译程序 | 可替换动态库        |
| 部署      | 简单      | 需处理库路径        |

## 5. 常用命令

```bash
# 生成目标文件
gcc -c file.c -o file.o

# 创建静态库
ar rcs libxxx.a file1.o file2.o

# 查看静态库内容
ar t libxxx.a

# 链接静态库
gcc main.c -L/path/to/lib -lxxx -o app

# 查看符号
nm libxxx.a
```

## 6. 常见问题

**链接顺序很重要：**

```bash
gcc main.o -L. -ladd -o main
```

通常库要放在使用它的目标文件后面。

**多个库有依赖时，也要注意顺序：**

```bash
gcc main.o -lA -lB -o main
```

如果 `A` 依赖 `B`，一般 `-lA` 要放在 `-lB` 前面。

简单理解：
**静态库像是把工具零件直接装进机器里；动态库像是机器运行时再去工具箱里拿工具。**