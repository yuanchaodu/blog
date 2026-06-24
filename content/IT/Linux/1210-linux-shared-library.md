---
title: Linux 动态库
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnVCF
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/121'
---

# Linux 动态库

<img src="images/Linux.svg" width="300">

Linux 动态库（Dynamic Library）是指程序运行时才加载到内存中的共享代码库。在 Linux 中，动态库文件通常以 **`.so`（Shared Object）** 为后缀。

例如：

```bash
libc.so
libpthread.so
libssl.so
```

其中，Linux 系统中的很多程序都会共享使用这些动态库。

## 一、动态库与静态库的区别

| 对比项     | 动态库（.so）     | 静态库（.a）    |
| ------- | ------------ | ---------- |
| 链接时间    | 程序运行时加载      | 编译时直接打包进程序 |
| 可执行文件大小 | 较小           | 较大         |
| 内存占用    | 多个程序共享       | 每个程序独立占用   |
| 更新库文件   | 无需重新编译程序（通常） | 需要重新编译     |
| 部署复杂度   | 需要依赖库文件      | 部署简单       |

简单理解：

* 静态库像把一本工具书复印后放进每个人的包里。
* 动态库像图书馆，大家需要时去借阅同一本书。

---

## 二、动态库的工作过程

假设程序：

```bash
hello
```

依赖：

```bash
libtest.so
```

运行时：

```text
hello
  │
  ├── 动态链接器(ld-linux)
  │
  └── libtest.so
```

程序启动时，动态链接器负责：

1. 查找依赖库
2. 加载到内存
3. 完成符号解析
4. 开始执行程序

---

## 三、查看程序依赖的动态库

使用：

```bash
ldd hello
```

示例：

```bash
$ ldd hello

linux-vdso.so.1
libtest.so => /usr/lib/libtest.so
libc.so.6 => /lib64/libc.so.6
```

表示程序运行时需要这些库。

---

## 四、创建动态库

### 1. 编写源文件

test.c

```c
#include <stdio.h>

void hello()
{
    printf("Hello Library\n");
}
```

---

### 2. 编译为位置无关代码

```bash
gcc -fPIC -c test.c
```

生成：

```bash
test.o
```

其中：

```text
-fPIC
```

表示 Position Independent Code（位置无关代码）。

---

### 3. 生成动态库

```bash
gcc -shared -o libtest.so test.o
```

生成：

```bash
libtest.so
```

---

## 五、使用动态库

main.c

```c
void hello();

int main()
{
    hello();
    return 0;
}
```

编译：

```bash
gcc main.c -L. -ltest -o hello
```

参数说明：

```text
-L.
```

指定库搜索路径。

```text
-ltest
```

表示链接：

```text
libtest.so
```

---

## 六、动态库搜索路径

Linux 会按顺序查找动态库：

### 1. LD_LIBRARY_PATH

```bash
export LD_LIBRARY_PATH=/opt/lib:$LD_LIBRARY_PATH
```

临时生效。

---

### 2. /etc/ld.so.conf

配置：

```bash
/etc/ld.so.conf
/etc/ld.so.conf.d/*.conf
```

更新缓存：

```bash
sudo ldconfig
```

---

### 3. 系统默认目录

常见目录：

```text
/lib
/lib64
/usr/lib
/usr/lib64
```

---

## 七、查看动态库信息

### 查看导出符号

```bash
nm -D libtest.so
```

或

```bash
objdump -T libtest.so
```

---

### 查看库依赖

```bash
ldd libtest.so
```

---

### 查看 ELF 信息

```bash
readelf -d libtest.so
```

---

## 八、运行时动态加载

除了启动时加载，还可以在程序运行过程中加载：

```c
#include <dlfcn.h>

void *handle = dlopen("./libtest.so", RTLD_LAZY);

void (*func)();

func = dlsym(handle, "hello");

func();

dlclose(handle);
```

编译：

```bash
gcc main.c -ldl
```

这种方式常用于：

* 插件系统
* 驱动扩展
* 热更新模块
* 可选功能加载

例如：

* 数据库插件
* 浏览器插件
* 工业软件功能模块

---

## 九、企业应用中的典型场景

对于工业企业服务器或数字化系统，经常会遇到：

### 中间件

* Nginx
* MySQL
* Oracle Client
* OpenSSL

都大量依赖动态库。

### 自研系统

例如：

```text
MES系统
├── 主程序
├── 数据库访问库
├── OPC通讯库
├── 日志库
└── 算法库
```

将公共功能封装成 `.so` 文件后：

* 代码复用更方便
* 升级更灵活
* 系统维护成本更低

---

## 十、一张图理解

```text
            编译阶段
                │
                ▼
          libtest.so
                │
                ▼
      hello（只记录依赖）
                │
          程序启动
                │
                ▼
       动态链接器加载
                │
                ▼
      libtest.so映射到内存
                │
                ▼
          调用函数执行
```

一句话概括：

**Linux 动态库（.so）本质上是一组可被多个程序共享、在运行时加载的代码文件，它能够减小程序体积、提高代码复用率，并简化系统升级维护。**
