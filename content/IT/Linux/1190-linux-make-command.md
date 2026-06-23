---
title: Linux make 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnUYf
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/119'
---

# Linux make 命令

<img src="images/Linux.svg" width="300">

`make` 是 Linux 中常用的**自动化构建工具**。它通常用来编译程序，也可以用来自动执行一组命令。

## 1. make 的作用

简单说，`make` 会根据一个叫 **Makefile** 的文件，判断哪些文件需要重新生成，然后执行对应命令。

常见用途：

```bash
make
```

编译项目。

```bash
make install
```

安装编译好的程序。

```bash
make clean
```

清理编译产生的文件。

## 2. Makefile 是什么

`Makefile` 是 `make` 的配置文件，里面写了“目标、依赖、命令”。

基本格式：

```makefile
目标: 依赖文件
	要执行的命令
```

注意：命令前面必须是 **Tab 键**，不能用普通空格。

## 3. 简单例子

假设有一个 C 文件：

```c
// hello.c
#include <stdio.h>

int main() {
    printf("Hello Linux\n");
    return 0;
}
```

写一个 `Makefile`：

```makefile
hello: hello.c
	gcc hello.c -o hello

clean:
	rm -f hello
```

执行：

```bash
make
```

会生成可执行文件：

```bash
./hello
```

清理文件：

```bash
make clean
```

## 4. make 的工作逻辑

`make` 会比较文件时间。

例如：

```makefile
hello: hello.c
	gcc hello.c -o hello
```

意思是：

如果 `hello.c` 比 `hello` 新，或者 `hello` 不存在，就重新编译。

如果 `hello` 已经是最新的，执行 `make` 时可能会提示：

```bash
make: 'hello' is up to date.
```

## 5. 常见命令

```bash
make
```

执行默认目标，通常是 Makefile 中第一个目标。

```bash
make clean
```

执行 `clean` 目标，清理临时文件。

```bash
make install
```

安装程序，通常需要管理员权限：

```bash
sudo make install
```

```bash
make -j4
```

使用 4 个并行任务，加快编译速度。

```bash
make -n
```

只显示将要执行的命令，不真正执行。

## 6. 常见 Makefile 示例

```makefile
CC = gcc
CFLAGS = -Wall -O2

hello: hello.o
	$(CC) hello.o -o hello

hello.o: hello.c
	$(CC) $(CFLAGS) -c hello.c

clean:
	rm -f hello hello.o
```

这里：

`CC = gcc` 表示编译器。

`CFLAGS = -Wall -O2` 表示编译参数。

`$(CC)` 表示引用变量。

## 7. 一句话理解

`make` 就像一个“施工队长”。
`Makefile` 是施工图纸。
你执行 `make`，它会按图纸判断哪些地方需要施工，然后只做必要的工作。
