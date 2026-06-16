---
title: Linux ACL 权限
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnKU0
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/77'
---

# Linux ACL 权限

<img src="images/Linux.svg" width="300">

Linux ACL 是 **Linux 访问控制列表**，用来补充传统的 `rwx` 权限。

传统权限只能控制三类对象：

```bash
所有者 user
所属组 group
其他人 others
```

ACL 可以做到更细，比如：

```bash
让张三能读写某个文件
让李四只能读
让某个部门组可以执行
```

## 1. 查看 ACL 权限

```bash
getfacl 文件名
```

示例：

```bash
getfacl test.txt
```

可能输出：

```bash
user::rw-
user:zhangsan:rw-
group::r--
mask::rw-
other::---
```

含义：

```bash
user::rw-              文件所有者权限
user:zhangsan:rw-      指定用户 zhangsan 的权限
group::r--             所属组权限
mask::rw-              ACL 最大有效权限
other::---             其他人权限
```

## 2. 设置用户 ACL 权限

给用户 `zhangsan` 添加读写权限：

```bash
setfacl -m u:zhangsan:rw test.txt
```

给用户只读权限：

```bash
setfacl -m u:zhangsan:r test.txt
```

取消用户权限：

```bash
setfacl -x u:zhangsan test.txt
```

## 3. 设置组 ACL 权限

给组 `itgroup` 添加读写权限：

```bash
setfacl -m g:itgroup:rw test.txt
```

取消组权限：

```bash
setfacl -x g:itgroup test.txt
```

## 4. 递归设置目录 ACL

对目录及其下面所有文件设置：

```bash
setfacl -R -m u:zhangsan:rw /data/project
```

注意：
`-R` 会影响已有文件和子目录。

## 5. 默认 ACL

默认 ACL 主要用于目录，表示以后新建的文件或目录自动继承权限。

给目录设置默认权限：

```bash
setfacl -m d:u:zhangsan:rw /data/project
```

查看：

```bash
getfacl /data/project
```

可能看到：

```bash
default:user:zhangsan:rw-
```

## 6. 删除 ACL

删除某个用户 ACL：

```bash
setfacl -x u:zhangsan test.txt
```

删除所有扩展 ACL：

```bash
setfacl -b test.txt
```

递归删除目录下所有 ACL：

```bash
setfacl -R -b /data/project
```

## 7. mask 的作用

`mask` 是 ACL 的最大有效权限。

例如：

```bash
user:zhangsan:rwx
mask::r--
```

实际生效权限只有：

```bash
r--
```

可以手动修改 mask：

```bash
setfacl -m m:rw test.txt
```

## 8. 常用命令汇总

```bash
getfacl file
setfacl -m u:user:rw file
setfacl -m g:group:r file
setfacl -x u:user file
setfacl -b file
setfacl -R -m u:user:rw directory
setfacl -m d:u:user:rw directory
```

## 简单理解

传统权限像“三把钥匙”：所有者、组、其他人。
ACL 像“门禁卡系统”，可以给不同人、不同组单独设置权限，更灵活。
