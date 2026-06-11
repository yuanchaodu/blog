---
title: Linux mv 命令
section: IT
category: Linux
---

# Linux mv 命令

<img src="images/Linux.svg" width="300">
`mv`（move）是 Linux 中用于**移动文件或目录**，以及**重命名文件或目录**的命令。

## 基本语法

```bash
mv [选项] 源文件 目标文件
```

或者

```bash
mv [选项] 源文件 源文件... 目标目录
```

---

## 1. 重命名文件

将 `file1.txt` 改名为 `file2.txt`：

```bash
mv file1.txt file2.txt
```

执行前：

```text
file1.txt
```

执行后：

```text
file2.txt
```

---

## 2. 移动文件

将文件移动到指定目录：

```bash
mv test.txt /home/user/docs/
```

移动后：

```text
/home/user/docs/test.txt
```

---

## 3. 移动多个文件

```bash
mv file1.txt file2.txt file3.txt /backup/
```

将三个文件一起移动到 `/backup/` 目录。

---

## 4. 移动目录

```bash
mv mydir /data/
```

把整个目录 `mydir` 移动到 `/data/` 下。

移动后：

```text
/data/mydir
```

---

## 5. 常用选项

### `-i`：覆盖前询问

如果目标文件已存在，会提示确认：

```bash
mv -i test.txt /backup/
```

输出示例：

```text
mv: overwrite '/backup/test.txt'?
```

输入：

```text
y
```

确认覆盖。

---

### `-f`：强制覆盖

不进行任何提示：

```bash
mv -f test.txt /backup/
```

即使目标文件存在，也直接覆盖。

---

### `-n`：不覆盖已有文件

```bash
mv -n test.txt /backup/
```

如果目标文件存在，则不执行移动。

---

### `-v`：显示详细过程

```bash
mv -v test.txt /backup/
```

输出：

```text
renamed 'test.txt' -> '/backup/test.txt'
```

适合脚本调试和查看执行结果。

---

## 6. 使用通配符批量移动

移动所有 `.log` 文件：

```bash
mv *.log /backup/logs/
```

移动所有图片：

```bash
mv *.jpg *.png /images/
```

---

## 7. 移动并改名

```bash
mv report.txt /backup/report_20250611.txt
```

既完成移动，又完成重命名。

---

## 8. 跨文件系统移动

例如：

```bash
mv /data/file.txt /mnt/usb/
```

如果源和目标位于不同文件系统：

1. 先复制文件
2. 再删除源文件

效果类似：

```bash
cp /data/file.txt /mnt/usb/
rm /data/file.txt
```

因此大文件跨磁盘移动时可能耗时较长。

---

## 9. 查看帮助

```bash
mv --help
```

查看手册：

```bash
man mv
```

---

## 常见示例汇总

```bash
# 重命名
mv old.txt new.txt

# 移动文件
mv test.txt /tmp/

# 移动多个文件
mv *.txt /backup/

# 覆盖前询问
mv -i file.txt /data/

# 强制覆盖
mv -f file.txt /data/

# 不覆盖已有文件
mv -n file.txt /data/

# 显示详细过程
mv -v file.txt /data/

# 移动并改名
mv report.txt /backup/report_bak.txt
```

## 注意事项

* `mv` 默认会覆盖同名文件（部分发行版可能配置了 `alias mv='mv -i'`）。
* 移动目录时不需要 `-r` 参数（与 `cp` 不同）。
* 跨文件系统移动大文件时，本质上是“复制+删除”，速度受磁盘性能影响。
* 使用 `mv -i` 可以避免误覆盖重要文件，是日常运维中比较推荐的习惯。