---
title: Linux 权限简介
section: IT
category: Linux
---

# Linux 权限简介

<img src="images/Linux.svg" width="300">

Linux 权限是 Linux 系统安全管理的基础机制，用于控制用户对文件和目录的访问能力。简单来说，它决定了“谁可以对什么资源做什么操作”。

## 一、Linux 用户体系

Linux 主要有三类身份：

### 1. Root 用户

Root 是系统管理员，拥有最高权限，可以访问和修改系统中的所有文件。

例如：

```bash
root
```

### 2. 普通用户

日常登录和工作使用的用户。

例如：

```bash
nick
tom
alice
```

普通用户只能访问自己有权限访问的资源。

### 3. 用户组（Group）

为了方便管理多个用户，Linux 引入了用户组。

例如：

```text
开发组（dev）
运维组（ops）
财务组（finance）
```

用户可以同时属于多个组。

查看当前用户信息：

```bash
id
```

输出示例：

```bash
uid=1000(nick) gid=1000(nick) groups=1000(nick),1001(dev)
```

---

## 二、文件权限的组成

查看文件权限：

```bash
ls -l
```

例如：

```bash
-rwxr-xr--
```

可以分为四部分：

```text
- rwx r-x r--
│ │   │   │
│ │   │   └── 其他用户权限
│ │   └────── 用户组权限
│ └────────── 文件所有者权限
└──────────── 文件类型
```

---

## 三、文件类型

第一位表示文件类型。

常见类型：

| 标识 | 类型     |
| -- | ------ |
| -  | 普通文件   |
| d  | 目录     |
| l  | 软链接    |
| c  | 字符设备   |
| b  | 块设备    |
| p  | 管道     |
| s  | Socket |

例如：

```text
-rw-r--r-- file.txt
drwxr-xr-x testdir
lrwxrwxrwx link
```

---

## 四、三种基本权限

Linux 权限用：

```text
r w x
```

表示。

### 1. r（Read，读）

数值：

```text
4
```

允许查看内容。

文件：

```bash
cat file.txt
```

目录：

```bash
ls dir/
```

---

### 2. w（Write，写）

数值：

```text
2
```

允许修改内容。

文件：

```bash
echo "hello" >> file.txt
```

目录：

```bash
touch newfile
rm oldfile
```

---

### 3. x（Execute，执行）

数值：

```text
1
```

允许执行程序。

例如：

```bash
./run.sh
```

对于目录来说：

```text
x = 允许进入目录
```

例如：

```bash
cd mydir
```

---

## 五、权限作用对象

Linux 对三类对象分别授权：

| 缩写 | 含义          |
| -- | ----------- |
| u  | User（所有者）   |
| g  | Group（所属组）  |
| o  | Other（其他用户） |

例如：

```text
-rwxr-xr--
```

表示：

| 对象  | 权限  |
| --- | --- |
| 所有者 | rwx |
| 用户组 | r-x |
| 其他人 | r-- |

---

## 六、数字权限表示法

Linux 常用数字表示权限：

| 权限 | 数值 |
| -- | -- |
| r  | 4  |
| w  | 2  |
| x  | 1  |

相加即可得到权限值。

### 常见组合

| 权限  | 数字 |
| --- | -- |
| rwx | 7  |
| rw- | 6  |
| r-x | 5  |
| r-- | 4  |
| --- | 0  |

例如：

```text
rwxr-xr--
```

转换：

```text
rwx = 7
r-x = 5
r-- = 4
```

即：

```bash
754
```

---

## 七、chmod 修改权限

### 数字方式

```bash
chmod 755 test.sh
```

结果：

```text
rwxr-xr-x
```

---

### 符号方式

增加执行权限：

```bash
chmod +x test.sh
```

等价于：

```bash
chmod a+x test.sh
```

---

给用户组增加写权限：

```bash
chmod g+w file.txt
```

---

移除其他用户权限：

```bash
chmod o-rwx file.txt
```

---

## 八、chown 修改所有者

查看文件：

```bash
ls -l
```

例如：

```text
-rw-r--r-- 1 root root file.txt
```

第一个 root：

```text
文件所有者
```

第二个 root：

```text
所属组
```

修改所有者：

```bash
chown nick file.txt
```

---

同时修改所有者和组：

```bash
chown nick:dev file.txt
```

---

递归修改目录：

```bash
chown -R nick:dev project/
```

---

## 九、目录权限与文件权限的区别

很多初学者容易混淆。

### 文件权限

| 权限 | 含义   |
| -- | ---- |
| r  | 查看内容 |
| w  | 修改内容 |
| x  | 执行文件 |

---

### 目录权限

| 权限 | 含义      |
| -- | ------- |
| r  | 查看目录列表  |
| w  | 创建、删除文件 |
| x  | 进入目录    |

例如：

目录权限：

```text
drwx------
```

拥有：

```text
rwx
```

则可以：

```bash
ls
cd
touch
rm
```

---

## 十、特殊权限

### SUID（Set User ID）

权限位：

```text
s
```

示例：

```bash
-rwsr-xr-x
```

当普通用户执行该程序时，会临时获得文件所有者的权限。

典型例子：

```bash
/usr/bin/passwd
```

普通用户修改密码时，需要写入系统密码文件，因此借助 SUID 完成。

---

### SGID（Set Group ID）

目录设置：

```bash
chmod g+s shared/
```

效果：

```text
新建文件自动继承目录所属组
```

适合团队共享目录。

---

### Sticky Bit（粘滞位）

典型目录：

```bash
/tmp
```

权限：

```text
drwxrwxrwt
```

含义：

```text
任何人可以创建文件，
但只能删除自己的文件。
```

---

## 十一、权限查看常用命令

查看当前用户：

```bash
whoami
```

查看用户身份：

```bash
id
```

查看文件权限：

```bash
ls -l
```

查看目录权限：

```bash
ls -ld 目录名
```

查看 umask：

```bash
umask
```

---

## 十二、企业环境中的常见权限设置

### Web 服务目录

```text
网站管理员：读写
Web服务账户：只读
其他用户：无权限
```

例如：

```bash
chmod 750 /var/www/html
```

---

### 共享项目目录

```bash
chmod 2775 project/
```

其中：

```text
2 = SGID
775 = rwxrwxr-x
```

团队成员创建的文件自动继承项目组。

---

### 配置文件

例如：

```bash
/etc/ssh/sshd_config
```

通常设置：

```bash
chmod 600
```

表示：

```text
仅所有者可读写
```

---

## 十三、快速记忆

可以把 Linux 权限理解为一个“三层门禁系统”：

```text
            文件/目录
                 │
    ┌────────────┼────────────┐
    │            │            │
 所有者        用户组       其他人
    │            │            │
  rwx          rwx          rwx
```

每个身份都分别拥有：

* r（读）
* w（写）
* x（执行）

三种权限的组合。

最常见的权限数字：

| 权限  | 数字      | 场景          |
| --- | ------- | ----------- |
| 777 | 全开放     | 测试环境（不推荐生产） |
| 755 | 程序、目录   | 最常用         |
| 644 | 普通文件    | 最常用         |
| 600 | 密钥、配置文件 | 安全要求高       |
| 700 | 用户私有脚本  | 仅本人可访问      |

对于日常运维和系统管理来说，理解 **755、644、600、700** 这几个权限组合，以及 **chmod、chown、ls -l** 三个命令，就已经掌握了 Linux 权限管理的核心内容。