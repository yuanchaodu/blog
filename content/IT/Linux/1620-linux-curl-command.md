---
title: Linux curl 命令
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnusK
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/162'
---

# Linux curl 命令

<img src="images/Linux.svg" width="300">

`curl`（Client URL）是 Linux 中最常用的网络命令之一，用于通过 URL 与服务器通信。它支持 HTTP、HTTPS、FTP、SFTP 等多种协议，广泛用于接口测试、下载文件、上传文件、自动化脚本等场景。

## 一、curl 的基本语法

```bash
curl [选项] URL
```

例如：

```bash
curl https://example.com
```

会输出网页的 HTML 内容。

---

## 二、常用用法

### 1. 获取网页内容

```bash
curl https://example.com
```

输出：

```html
<!doctype html>
<html>
...
```

---

### 2. 下载文件

保持原文件名：

```bash
curl -O https://example.com/file.zip
```

下载并指定文件名：

```bash
curl -o myfile.zip https://example.com/file.zip
```

区别：

| 参数   | 说明         |
| ---- | ---------- |
| `-O` | 使用服务器上的文件名 |
| `-o` | 自己指定文件名    |

---

### 3. 查看 HTTP 响应头

```bash
curl -I https://example.com
```

输出例如：

```text
HTTP/2 200
content-type: text/html
server: nginx
...
```

常用于检查：

* HTTP 状态码
* Content-Type
* Server
* Cache-Control

---

### 4. 显示完整请求和响应

```bash
curl -v https://example.com
```

`-v`（verbose）会显示：

* DNS 解析
* TCP 建立
* SSL 握手
* HTTP 请求头
* HTTP 响应头

排查 HTTPS、代理或连接问题时非常有用。

---

### 5. 只查看 HTTP 状态码

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://example.com
```

输出：

```text
200
```

常用于脚本判断网站是否正常。

---

### 6. 跟随重定向

默认不会自动跳转。

例如：

```bash
curl https://github.com
```

如果返回 301/302，可加：

```bash
curl -L https://github.com
```

`-L` 表示自动跟随重定向。

---

## 三、发送 GET 请求

带参数：

```bash
curl "https://example.com/api?id=100&name=test"
```

也可以：

```bash
curl -G \
     -d "id=100" \
     -d "name=test" \
     https://example.com/api
```

最终请求：

```text
GET /api?id=100&name=test
```

---

## 四、发送 POST 请求

### 表单（application/x-www-form-urlencoded）

```bash
curl -X POST \
     -d "username=admin" \
     -d "password=123456" \
     https://example.com/login
```

等价于：

```
username=admin&password=123456
```

---

### JSON 请求（最常见）

```bash
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name":"Tom","age":18}' \
     https://example.com/api/user
```

其中：

* `-H`：添加请求头
* `-d`：请求体

---

### 从文件读取 JSON

```bash
curl -X POST \
     -H "Content-Type: application/json" \
     -d @data.json \
     https://example.com/api
```

data.json：

```json
{
  "name":"Tom",
  "age":18
}
```

---

## 五、发送 PUT 请求

```bash
curl -X PUT \
     -H "Content-Type: application/json" \
     -d '{"name":"Jerry"}' \
     https://example.com/api/user/1
```

---

## 六、发送 DELETE 请求

```bash
curl -X DELETE \
     https://example.com/api/user/1
```

---

## 七、添加请求头

例如：

```bash
curl \
    -H "Authorization: Bearer xxxxxxxxx" \
    -H "Accept: application/json" \
    https://example.com/api
```

多个 Header：

```bash
-H "key:value"
-H "key2:value2"
```

---

## 八、Basic Authentication

```bash
curl -u username:password https://example.com
```

等价于：

```
Authorization: Basic xxxxxx
```

---

## 九、Bearer Token

REST API 中最常见：

```bash
curl \
-H "Authorization: Bearer eyJhbGc..." \
https://example.com/api
```

---

## 十、上传文件

### multipart/form-data

```bash
curl \
-F "file=@test.pdf" \
https://example.com/upload
```

多个文件：

```bash
curl \
-F "file1=@a.txt" \
-F "file2=@b.txt" \
https://example.com/upload
```

---

## 十一、下载并显示进度

```bash
curl -# -O https://example.com/file.iso
```

或

```bash
curl --progress-bar -O https://example.com/file.iso
```

---

## 十二、断点续传

```bash
curl -C - -O https://example.com/file.iso
```

适用于大文件下载中断后的恢复。

---

## 十三、指定代理

```bash
curl -x http://127.0.0.1:7890 https://google.com
```

SOCKS5：

```bash
curl --socks5 127.0.0.1:1080 https://example.com
```

---

## 十四、忽略 HTTPS 证书

```bash
curl -k https://example.com
```

或：

```bash
curl --insecure https://example.com
```

仅建议在测试环境使用。

---

## 十五、保存 Cookie

```bash
curl -c cookie.txt https://example.com
```

读取 Cookie：

```bash
curl -b cookie.txt https://example.com
```

---

## 十六、设置超时

连接超时：

```bash
curl --connect-timeout 5 https://example.com
```

整个请求超时：

```bash
curl --max-time 10 https://example.com
```

---

## 十七、限制下载速度

```bash
curl --limit-rate 1M -O https://example.com/file.iso
```

限制为：

```
1 MB/s
```

---

## 十八、指定 User-Agent

```bash
curl \
-A "Mozilla/5.0" \
https://example.com
```

或：

```bash
curl --user-agent "Chrome"
```

---

## 十九、常用参数速查

| 参数                  | 作用                                |
| ------------------- | --------------------------------- |
| `-X`                | 指定 HTTP 方法（GET、POST、PUT、DELETE 等） |
| `-H`                | 添加请求头                             |
| `-d`                | 提交请求数据                            |
| `-F`                | 上传表单或文件（multipart/form-data）      |
| `-o`                | 保存为指定文件名                          |
| `-O`                | 使用服务器文件名保存                        |
| `-I`                | 只获取响应头                            |
| `-v`                | 显示详细调试信息                          |
| `-L`                | 跟随重定向                             |
| `-u`                | Basic Authentication              |
| `-k`                | 忽略 HTTPS 证书校验                     |
| `-c`                | 保存 Cookie                         |
| `-b`                | 发送 Cookie                         |
| `-A`                | 指定 User-Agent                     |
| `-s`                | 静默模式，不显示进度                        |
| `-w`                | 自定义输出格式（如状态码、耗时等）                 |
| `--connect-timeout` | 设置连接超时                            |
| `--max-time`        | 设置整个请求超时                          |
| `-C -`              | 断点续传                              |
| `--limit-rate`      | 限制传输速率                            |

## 二十、实际应用示例

**1. 检查网站是否可访问：**

```bash
curl -o /dev/null -s -w "HTTP %{http_code}\n" https://example.com
```

**2. 调用 REST API：**

```bash
curl -X POST \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"device":"server01","status":"online"}' \
  https://api.example.com/devices
```

**3. 下载软件安装包：**

```bash
curl -L -O https://example.com/software.tar.gz
```

**4. 查看服务器响应头：**

```bash
curl -I https://example.com
```

对于日常 Linux 运维、接口测试和自动化脚本，熟练掌握 `-H`、`-d`、`-X`、`-o/-O`、`-L`、`-I`、`-v`、`-s`、`-w` 这几个参数，就可以覆盖绝大多数使用场景。
