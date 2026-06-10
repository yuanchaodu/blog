---
title: Linux mkdir 命令
section: IT
category: Linux
---

# Linux mkdir 命令

<img src="images/Linux.svg" width="300">


`mkdir`（make directory）是 Linux 中用于**创建目录（文件夹）**的命令。

## 基本语法

```bash
mkdir [选项] 目录名
```

## 常见用法

### 1. 创建单个目录

```bash
mkdir test
```

创建一个名为 `test` 的目录。

---

### 2. 创建多个目录

```bash
mkdir dir1 dir2 dir3
```

一次创建多个目录：

```text
dir1
dir2
dir3
```

---

### 3. 创建多级目录

使用 `-p` 参数：

```bash
mkdir -p project/src/java
```

如果 `project` 和 `src` 不存在，会自动创建。

创建结果：

```text
project
└── src
    └── java
```

如果目录已存在，也不会报错。

---

### 4. 创建目录并指定权限

使用 `-m` 参数：

```bash
mkdir -m 755 testdir
```

创建目录时直接设置权限。

对应权限：

```text
rwxr-xr-x
```

查看权限：

```bash
ls -ld testdir
```

---

### 5. 显示创建过程

使用 `-v` 参数：

```bash
mkdir -v test
```

输出：

```text
mkdir: created directory 'test'
```

---

## 常用选项

| 选项       | 说明      |
| -------- | ------- |
| `-p`     | 递归创建父目录 |
| `-m`     | 指定目录权限  |
| `-v`     | 显示创建过程  |
| `--help` | 查看帮助信息  |

---

## 实际工作示例

### 创建项目目录结构

```bash
mkdir -p /data/project/{bin,conf,log,tmp}
```

创建结果：

```text
project
├── bin
├── conf
├── log
└── tmp
```

---

### 创建按日期归档目录

```bash
mkdir -p backup/$(date +%Y/%m/%d)
```

假设今天是 2026 年 6 月 10 日，则生成：

```text
backup/
└── 2026
    └── 06
        └── 10
```

---

### 批量创建部门目录

```bash
mkdir 财务部 人事部 生产部 IT部
```

---

## 常见错误

### 错误 1：目录已存在

```bash
mkdir test
```

输出：

```text
mkdir: cannot create directory 'test': File exists
```

解决方法：

```bash
mkdir -p test
```

---

### 错误 2：权限不足

```bash
mkdir /root/test
```

输出：

```text
mkdir: cannot create directory '/root/test': Permission denied
```

解决方法：

```bash
sudo mkdir /root/test
```

---

## 查看帮助

```bash
man mkdir
```

或

```bash
mkdir --help
```

查看系统中的完整参数说明。