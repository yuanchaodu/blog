---
title: Linux 源码安装软件
section: IT
category: Linux
---

# Linux 源码安装软件

<img src="images/Linux.svg" width="300">

Linux 源码安装软件，一般就是把程序源代码下载下来，在本机编译成可执行文件，再安装到系统中。

常见流程如下：

```bash
# 1. 解压源码包
tar -zxvf software.tar.gz
cd software

# 2. 查看说明
ls
cat README
cat INSTALL

# 3. 安装编译工具和依赖
sudo apt install build-essential
# 或 CentOS/RHEL：
sudo yum groupinstall "Development Tools"

# 4. 配置编译参数
./configure --prefix=/usr/local/software

# 5. 编译
make

# 6. 安装
sudo make install
```

常见命令含义：

```bash
./configure
```

检查系统环境、依赖库，并生成 Makefile。

```bash
make
```

根据 Makefile 编译源码。

```bash
sudo make install
```

把编译好的程序复制到指定目录。

卸载时，如果源码目录还在，通常可以执行：

```bash
sudo make uninstall
```

但不是所有软件都支持。

建议使用：

```bash
./configure --prefix=/usr/local/软件名
```

这样安装路径清晰，后期删除也方便。例如：

```bash
sudo rm -rf /usr/local/软件名
```

实际工作中，更推荐优先使用系统包管理器安装：

```bash
sudo apt install 软件名
sudo yum install 软件名
```

只有在需要特定版本、官方仓库没有、或需要自定义编译参数时，再考虑源码安装。