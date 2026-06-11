---
title: Linux which 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnDU5
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/33'
---

# Linux which 命令

<img src="images/Linux.svg" width="300">

`which` 是 Linux 中用于**查找命令可执行文件所在路径**的工具。

### 基本语法

```bash
which 命令名
```

例如：

```bash
which ls
```

输出：

```bash
/usr/bin/ls
```

表示系统执行 `ls` 命令时，实际调用的是 `/usr/bin/ls` 文件。

---

### 常见用途

#### 1. 查看命令安装位置

```bash
which python3
```

可能输出：

```bash
/usr/bin/python3
```

#### 2. 查看多个命令的位置

```bash
which gcc make git
```

输出：

```bash
/usr/bin/gcc
/usr/bin/make
/usr/bin/git
```

#### 3. 查看所有匹配项

默认只显示第一个匹配结果。

使用 `-a` 参数查看所有匹配项：

```bash
which -a python
```

例如：

```bash
/usr/local/bin/python
/usr/bin/python
```

说明系统的 `PATH` 环境变量中存在多个名为 `python` 的可执行文件。

---

### 工作原理

Linux 执行命令时，会按照 `PATH` 环境变量中定义的目录顺序查找。

查看 PATH：

```bash
echo $PATH
```

例如：

```bash
/usr/local/bin:/usr/bin:/bin:/usr/sbin
```

当执行：

```bash
python3
```

系统会依次在这些目录中寻找 `python3`。

`which` 也是按照同样的规则进行查找。

---

### 与 `whereis` 的区别

| 命令        | 作用                   |
| --------- | -------------------- |
| `which`   | 查找命令实际执行路径           |
| `whereis` | 查找命令、源码、man 手册等相关文件  |
| `find`    | 在指定目录递归搜索文件          |
| `type`    | 判断命令类型（别名、内置命令、外部命令） |

例如：

```bash
whereis ls
```

输出：

```bash
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

---

### `which` 的局限性

对于 **Shell 内置命令（builtin）**，`which` 有时不能准确判断。

例如：

```bash
which cd
```

可能输出：

```bash
/usr/bin/which: no cd in (...)
```

因为 `cd` 是 Bash 内置命令，不是独立可执行文件。

这时推荐使用：

```bash
type cd
```

输出：

```bash
cd is a shell builtin
```

或者：

```bash
type ls
```

输出：

```bash
ls is /usr/bin/ls
```

---

### 实际运维中更推荐使用 `type`

很多 Linux 管理员和脚本开发人员更喜欢：

```bash
type 命令名
```

因为它不仅能显示路径，还能区分：

* Shell 内置命令（builtin）
* 别名（alias）
* 函数（function）
* 外部可执行文件

例如：

```bash
type python
type cd
type ll
```

能够得到比 `which` 更完整的信息。

因此在现代 Linux（尤其是 Bash 环境）中，**排查命令来源时优先使用 `type`，查看可执行文件路径时使用 `which`**。
