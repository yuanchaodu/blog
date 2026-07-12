---
title: Linux wget 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnulQ
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/161'
---

# Linux wget 命令

<img src="images/Linux.svg" width="300">

`wget` 是 Linux 中常用的命令行下载工具，可以通过 HTTP、HTTPS、FTP 等协议下载文件。

## 一、基本格式

```bash
wget [选项] 下载地址
```

例如：

```bash
wget https://example.com/file.zip
```

文件会下载到当前目录。

## 二、常用命令

### 1. 指定保存文件名

```bash
wget -O newname.zip https://example.com/file.zip
```

`-O` 后面是保存后的文件名。

### 2. 指定下载目录

```bash
wget -P /home/user/downloads https://example.com/file.zip
```

`-P` 用于指定保存目录。

### 3. 断点续传

```bash
wget -c https://example.com/file.zip
```

下载中断后，再次执行可从上次位置继续下载。

### 4. 后台下载

```bash
wget -b https://example.com/file.zip
```

下载日志默认保存在：

```bash
wget-log
```

查看日志：

```bash
tail -f wget-log
```

### 5. 限制下载速度

```bash
wget --limit-rate=1m https://example.com/file.zip
```

表示下载速度限制为每秒 1 MB。

### 6. 设置重试次数

```bash
wget --tries=5 https://example.com/file.zip
```

下载失败时最多重试 5 次。

### 7. 设置超时时间

```bash
wget --timeout=30 https://example.com/file.zip
```

超时时间为 30 秒。

### 8. 下载时不显示详细信息

```bash
wget -q https://example.com/file.zip
```

`-q` 表示安静模式。

### 9. 显示下载进度

```bash
wget --progress=bar https://example.com/file.zip
```

适合在终端中查看进度。

## 三、需要认证的网站

### 用户名和密码

```bash
wget --user=username --password=password https://example.com/file.zip
```

更安全的方式是避免直接把密码写在命令中，因为命令可能被保存在历史记录里。

## 四、忽略 HTTPS 证书检查

```bash
wget --no-check-certificate https://example.com/file.zip
```

这个选项只建议用于测试环境。正式环境中应检查证书问题，不要长期忽略证书验证。

## 五、批量下载

先建立一个文本文件，例如 `urls.txt`：

```text
https://example.com/file1.zip
https://example.com/file2.zip
https://example.com/file3.zip
```

执行：

```bash
wget -i urls.txt
```

## 六、下载整个网站

```bash
wget -r https://example.com
```

常见组合：

```bash
wget -r -np -k -p https://example.com
```

参数含义：

* `-r`：递归下载。
* `-np`：不进入上级目录。
* `-k`：把网页链接转换为本地可用链接。
* `-p`：下载网页需要的图片、样式等资源。

下载网站时应遵守网站许可、访问规则和版权要求。

## 七、通过代理下载

```bash
export http_proxy=http://192.168.1.10:8080
export https_proxy=http://192.168.1.10:8080

wget https://example.com/file.zip
```

取消代理：

```bash
unset http_proxy
unset https_proxy
```

## 八、查看帮助和版本

```bash
wget --help
```

```bash
wget --version
```

最常用的组合通常是：

```bash
wget -c -O app.zip https://example.com/app.zip
```

它表示：下载文件、支持断点续传，并保存为 `app.zip`。
