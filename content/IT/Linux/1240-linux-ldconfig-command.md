---
title: Linux ldconfig 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnVur
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/124'
---

# Linux ldconfig 命令

<img src="images/Linux.svg" width="300">

`ldconfig` 是 Linux 系统中用于**配置和维护动态链接库缓存**的命令。它的主要作用是扫描系统中的共享库（`.so` 文件），更新库文件的软链接，并生成动态链接器使用的缓存文件。

## 一、通俗理解

可以把 Linux 动态库看成一本本工具书。

当程序启动时，需要找到对应的动态库，例如：

```bash
libc.so.6
libstdc++.so.6
libssl.so.3
```

如果每次都在整个磁盘里搜索，效率会很低。

`ldconfig` 的作用就像图书馆管理员：

* 扫描所有图书（动态库）
* 建立索引（缓存）
* 更新目录（软链接）
* 方便程序快速找到需要的库

---

## 二、工作原理

### 1. 扫描库目录

默认扫描：

```bash
/lib
/usr/lib
/lib64
/usr/lib64
```

以及配置文件指定的目录：

```bash
/etc/ld.so.conf
/etc/ld.so.conf.d/*.conf
```

例如：

```bash
cat /etc/ld.so.conf
```

输出：

```text
include /etc/ld.so.conf.d/*.conf
```

查看具体配置：

```bash
ls /etc/ld.so.conf.d/
```

---

### 2. 更新软链接

例如某目录存在：

```text
libtest.so.1.2.0
```

执行：

```bash
ldconfig
```

可能自动生成：

```text
libtest.so.1 -> libtest.so.1.2.0
```

这样程序只需要寻找：

```text
libtest.so.1
```

即可找到实际库文件。

---

### 3. 生成缓存

缓存文件：

```bash
/etc/ld.so.cache
```

动态链接器启动程序时会优先读取：

```bash
/etc/ld.so.cache
```

而不是遍历整个文件系统。

---

## 三、常用命令

### 查看缓存中的库

```bash
ldconfig -p
```

输出示例：

```text
libssl.so.3 (libc6,x86-64) => /usr/lib64/libssl.so.3
libcrypto.so.3 (libc6,x86-64) => /usr/lib64/libcrypto.so.3
```

含义：

* 库名称
* 架构类型
* 实际路径

---

### 查询某个库

```bash
ldconfig -p | grep libssl
```

输出：

```text
libssl.so.3 => /usr/lib64/libssl.so.3
```

---

### 更新缓存

```bash
sudo ldconfig
```

安装新库后经常执行。

例如：

```bash
tar -xf xxx.tar.gz
cp libxxx.so /usr/local/lib

sudo ldconfig
```

否则程序可能提示：

```text
error while loading shared libraries:
libxxx.so: cannot open shared object file
```

---

### 指定目录

```bash
ldconfig /usr/local/lib
```

仅处理指定目录。

---

### 不更新缓存，仅检查

```bash
ldconfig -N
```

---

### 仅更新链接

```bash
ldconfig -X
```

不重建缓存。

---

## 四、典型应用场景

### 场景1：安装第三方库

例如安装 Oracle Client：

```bash
cp *.so /usr/local/lib/
sudo ldconfig
```

---

### 场景2：编译软件后找不到库

错误：

```text
error while loading shared libraries:
libabc.so.1: cannot open shared object file
```

检查：

```bash
ldconfig -p | grep libabc
```

如果找不到：

```bash
echo "/opt/abc/lib" > /etc/ld.so.conf.d/abc.conf
sudo ldconfig
```

---

### 场景3：查看系统库

```bash
ldconfig -p | less
```

查看系统当前识别到的所有共享库。

---

## 五、与 ldd 的区别

很多人容易混淆。

| 命令         | 作用          |
| ---------- | ----------- |
| `ldconfig` | 管理动态库缓存     |
| `ldd`      | 查看程序依赖哪些动态库 |

例如：

查看 nginx 依赖：

```bash
ldd /usr/sbin/nginx
```

输出：

```text
libpcre.so.1 => /usr/lib64/libpcre.so.1
libssl.so.3 => /usr/lib64/libssl.so.3
```

查看系统是否认识这些库：

```bash
ldconfig -p | grep libssl
```

---

## 六、常见故障排查

### 问题1：找不到共享库

错误：

```text
cannot open shared object file
```

排查：

```bash
ldconfig -p | grep 库名
```

若无结果：

```bash
find / -name "libxxx.so*" 2>/dev/null
```

找到后加入：

```bash
echo "/库所在目录" > /etc/ld.so.conf.d/xxx.conf
sudo ldconfig
```

---

### 问题2：库版本不匹配

例如：

```text
libstdc++.so.6: version `GLIBCXX_3.4.29' not found
```

查看实际加载：

```bash
ldconfig -p | grep libstdc++
```

再检查版本：

```bash
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX
```

---

### 问题3：缓存未更新

查看缓存时间：

```bash
ls -l /etc/ld.so.cache
```

重新生成：

```bash
sudo ldconfig
```

---

## 七、最常用的几个命令

```bash
# 更新动态库缓存
sudo ldconfig

# 查看所有已识别库
ldconfig -p

# 查找指定库
ldconfig -p | grep libssl

# 查看程序依赖
ldd 程序名

# 查看库配置目录
cat /etc/ld.so.conf
```

对于运维、开发和系统管理员来说，`ldconfig` 最常见的用途就是：**安装新的 `.so` 动态库后执行一次 `sudo ldconfig`，让系统能够立即识别并加载这些库。**
