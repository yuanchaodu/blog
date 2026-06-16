---
title: Linux umask 命令
section: IT
category: Linux
---

# Linux umask 命令

<img src="images/Linux.svg" width="300">

`umask`（User File Creation Mask，用户文件创建掩码）是 Linux/Unix 系统中用于控制**新创建文件和目录默认权限**的命令。

简单理解：

> `chmod` 是事后修改权限，`umask` 是提前规定新文件创建时不能拥有哪些权限。

---

# 一、查看当前 umask

```bash
umask
```

例如输出：

```bash
0022
```

或者：

```bash
u=rwx,g=rx,o=rx
```

查看符号形式：

```bash
umask -S
```

输出：

```bash
u=rwx,g=rx,o=rx
```

---

# 二、umask 的工作原理

Linux 创建文件和目录时有默认权限：

| 类型 | 默认权限            |
| -- | --------------- |
| 文件 | 666 (rw-rw-rw-) |
| 目录 | 777 (rwxrwxrwx) |

然后再减去 umask 指定的权限。

计算公式：

```text
最终权限 = 默认权限 - umask
```

实际上是按位屏蔽（mask）计算。

---

# 三、示例

## 示例1：umask=022

```bash
umask 022
```

### 创建文件

默认：

```text
666
```

减去：

```text
022
```

结果：

```text
644
```

即：

```text
rw-r--r--
```

### 创建目录

默认：

```text
777
```

减去：

```text
022
```

结果：

```text
755
```

即：

```text
rwxr-xr-x
```

验证：

```bash
touch test.txt
mkdir testdir

ls -l
```

结果：

```text
-rw-r--r-- test.txt
drwxr-xr-x testdir
```

---

## 示例2：umask=002

```bash
umask 002
```

创建权限：

文件：

```text
664
rw-rw-r--
```

目录：

```text
775
rwxrwxr-x
```

这种配置常用于：

* 开发团队
* 共享目录
* NFS共享

因为同组用户有写权限。

---

## 示例3：umask=077

```bash
umask 077
```

文件：

```text
600
rw-------
```

目录：

```text
700
rwx------
```

适用于：

* 安全要求高的服务器
* 个人私有数据目录

---

# 四、为什么文件不是 777？

很多人会疑惑：

```text
777 - 022 = 755
```

为什么文件默认不是 755？

原因是：

Linux 普通文件默认不赋予执行权限。

默认基准权限：

```text
文件：666
目录：777
```

因此：

```text
文件：
666 - 022 = 644

目录：
777 - 022 = 755
```

---

# 五、临时修改 umask

当前终端生效：

```bash
umask 027
```

查看：

```bash
umask
```

输出：

```text
0027
```

只影响：

* 当前 Shell
* 当前 Shell 启动的子进程

关闭终端后失效。

---

# 六、永久修改 umask

## 方式1：用户级

编辑：

```bash
~/.bashrc
```

或者：

```bash
~/.profile
```

增加：

```bash
umask 022
```

生效：

```bash
source ~/.bashrc
```

---

## 方式2：系统级

编辑：

```bash
/etc/profile
```

或者：

```bash
/etc/bashrc
```

增加：

```bash
umask 022
```

对所有用户生效。

---

# 七、常见 umask 配置

| umask | 文件权限 | 目录权限 | 场景         |
| ----- | ---- | ---- | ---------- |
| 022   | 644  | 755  | Linux 默认配置 |
| 002   | 664  | 775  | 团队协作       |
| 027   | 640  | 750  | 企业服务器      |
| 077   | 600  | 700  | 高安全场景      |

---

# 八、企业环境推荐

对于企业服务器（如化工企业 MES、ERP、数据采集服务器等），通常推荐：

```bash
umask 027
```

效果：

```text
文件：640
目录：750
```

权限说明：

```text
Owner  ：读写
Group  ：只读
Others ：无权限
```

这样既保证同组运维人员可访问，又避免其他用户查看敏感数据，是生产环境中较为常见的配置。

---

# 九、快速记忆

可以把 `umask` 理解为：

```text
777 = 最大权限
umask = 禁止权限
最终权限 = 默认权限 - 禁止权限
```

例如：

```bash
umask 022
```

表示：

```text
自己：不限制
同组：禁止写
其他：禁止写
```

因此新建目录变成：

```text
rwxr-xr-x (755)
```

新建文件变成：

```text
rw-r--r-- (644)
```

这样记忆和计算会比较直观。