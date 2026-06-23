---
title: Linux GCC 介绍
section: IT
category: Linux
---

# Linux GCC 介绍

<img src="images/Linux.svg" width="300">

## Linux GCC 简介

GCC 是 **GNU Compiler Collection**，中文常叫 **GNU 编译器套件**。它是 Linux 中最常用的编译工具之一，可以把 C、C++、Fortran、Go 等语言写的源代码，编译成计算机能运行的程序。GCC 官方最新主线版本为 **GCC 16.1，发布于 2026 年 4 月 30 日**。([GCC][1])

## GCC 在 Linux 中的作用

简单说，GCC 就像“翻译员”。程序员写的是人能看懂的代码，比如 C 语言；GCC 负责把它翻译成机器能执行的二进制程序。

例如：

```bash
gcc hello.c -o hello
./hello
```

含义是：把 `hello.c` 编译成名为 `hello` 的可执行文件，然后运行它。

## 常见命令

编译 C 程序：

```bash
gcc main.c -o main
```

编译 C++ 程序：

```bash
g++ main.cpp -o main
```

只生成目标文件：

```bash
gcc -c main.c
```

开启警告信息：

```bash
gcc -Wall main.c -o main
```

开启优化：

```bash
gcc -O2 main.c -o main
```

## 编译过程

GCC 编译程序大致分为四步：

1. **预处理**：处理 `#include`、`#define`。
2. **编译**：把源代码转成汇编代码。
3. **汇编**：把汇编代码转成目标文件。
4. **链接**：把多个目标文件和库文件合成可执行程序。

可以理解为：先整理材料，再翻译，再打包，最后组装成成品。

## 常见用途

GCC 常用于：

* 编译 Linux 下的 C/C++ 程序。
* 编译开源软件源码。
* 开发嵌入式程序。
* 构建系统工具、服务程序和驱动相关代码。
* 配合 Make、CMake 等工具完成大型项目编译。

## 小结

GCC 是 Linux 软件开发中的基础工具。只要涉及 C/C++ 程序编译，基本都会遇到它。对系统运维、软件编译、源码安装和嵌入式开发来说，掌握 GCC 是很实用的基础能力。

[1]: https://gcc.gnu.org "GCC, the GNU Compiler Collection - GNU Project"