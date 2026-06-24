---
title: Linux ldd 命令
section: IT
category: Linux
---

# Linux ldd 命令

<img src="images/Linux.svg" width="300">

`ldd`（List Dynamic Dependencies）是 Linux 中用于查看**可执行文件或共享库依赖了哪些动态链接库（.so 文件）**的命令。

简单来说，它可以帮助你判断一个程序运行时需要哪些动态库，以及这些库是否能够正确找到。

---

# 一、基本语法

```bash
ldd 文件名
```

例如：

```bash
ldd /bin/ls
```

输出类似：

```bash
linux-vdso.so.1 (0x00007fff5e1f0000)
libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f4e4d8e7000)
libc.so.6 => /lib64/libc.so.6 (0x00007f4e4d500000)
libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007f4e4d460000)
/lib64/ld-linux-x86-64.so.2 (0x00007f4e4d940000)
```

---

# 二、输出内容解释

以这一行为例：

```bash
libc.so.6 => /lib64/libc.so.6 (0x00007f4e4d500000)
```

含义：

| 部分                   | 说明        |
| -------------------- | --------- |
| libc.so.6            | 程序需要的库名称  |
| =>                   | 映射关系      |
| /lib64/libc.so.6     | 实际加载的库文件  |
| (0x00007f4e4d500000) | 加载到内存中的地址 |

---

# 三、常见用途

## 1. 查看程序依赖库

例如：

```bash
ldd myapp
```

查看：

* 是否依赖某个库
* 依赖哪些版本

---

## 2. 排查库缺失问题

例如：

```bash
ldd myapp
```

输出：

```bash
libssl.so.1.1 => not found
```

说明：

程序需要：

```text
libssl.so.1.1
```

但系统中找不到。

这是很多程序启动报错的原因：

```bash
error while loading shared libraries
```

---

## 3. 检查部署环境

例如把程序从开发环境拷贝到生产环境后：

```bash
ldd myapp
```

检查：

```bash
not found
```

的库是否存在。

---

# 四、查看共享库的依赖关系

不仅可以查看可执行程序：

```bash
ldd libxxx.so
```

例如：

```bash
ldd libmysqlclient.so
```

输出：

```bash
libpthread.so.0 => /lib64/libpthread.so.0
libdl.so.2 => /lib64/libdl.so.2
libc.so.6 => /lib64/libc.so.6
```

说明该库又依赖其他库。

---

# 五、常用参数

## -v

显示更详细信息

```bash
ldd -v myapp
```

输出：

```bash
Version information:
...
```

可查看：

* 库版本要求
* 符号版本

常用于版本兼容性分析。

---

## -u

显示未使用的依赖库

```bash
ldd -u myapp
```

例如：

```bash
Unused direct dependencies:
libpthread.so.0
```

说明链接了该库但实际上没使用。

---

## -r

检查所有重定位

```bash
ldd -r myapp
```

会检查：

* 函数符号
* 数据符号

是否能够正确解析。

例如：

```bash
undefined symbol: SSL_CTX_new
```

用于排查运行时链接问题。

---

## -d

检查数据重定位

```bash
ldd -d myapp
```

只检查数据符号。

---

# 六、典型案例

## 案例1：排查程序启动失败

程序：

```bash
./myapp
```

报错：

```bash
error while loading shared libraries:
libcrypto.so.1.1: cannot open shared object file
```

执行：

```bash
ldd myapp
```

结果：

```bash
libcrypto.so.1.1 => not found
```

说明缺少 OpenSSL 库。

---

## 案例2：查看 nginx 依赖

```bash
ldd $(which nginx)
```

可能看到：

```bash
libpcre.so.1
libssl.so.1.1
libcrypto.so.1.1
libz.so.1
```

说明 nginx 依赖：

* PCRE
* OpenSSL
* zlib

---

## 案例3：查看 Java 依赖

```bash
ldd $(which java)
```

输出：

```bash
libpthread.so.0
libdl.so.2
libc.so.6
libstdc++.so.6
```

说明 JVM 本身也是一个 ELF 可执行程序。

---

# 七、ldd 的实现原理

实际上：

```bash
ldd
```

并不是直接解析 ELF 文件。

本质上通常会调用动态链接器：

```bash
/lib64/ld-linux-x86-64.so.2
```

并设置环境变量：

```bash
LD_TRACE_LOADED_OBJECTS=1
```

例如：

```bash
LD_TRACE_LOADED_OBJECTS=1 ./myapp
```

效果与：

```bash
ldd myapp
```

基本一致。

---

# 八、安全注意事项

对于**来源不可信的可执行文件**，不要直接运行：

```bash
ldd unknown_binary
```

原因是某些系统上的 `ldd` 实现可能会实际执行目标程序，从而带来安全风险。

更安全的方法是使用：

```bash
readelf -d 文件名
```

或：

```bash
objdump -p 文件名
```

查看依赖库。

例如：

```bash
readelf -d myapp | grep NEEDED
```

输出：

```bash
0x0000000000000001 (NEEDED) Shared library: [libssl.so.1.1]
0x0000000000000001 (NEEDED) Shared library: [libcrypto.so.1.1]
```

---

# 九、与相关命令比较

| 命令         | 用途           |
| ---------- | ------------ |
| ldd        | 查看动态库依赖      |
| readelf -d | 查看 ELF 动态段   |
| objdump -p | 查看 ELF 信息和依赖 |
| nm         | 查看符号表        |
| strings    | 查看文件中的字符串    |
| file       | 查看文件类型       |

在实际运维和软件部署中，最常见的排障组合是：

```bash
file myapp
ldd myapp
readelf -d myapp
```

这三个命令基本可以快速定位 80% 以上的动态库加载问题。