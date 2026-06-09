---
title: Linux 安装指南
section: IT
category: Linux
discussion_id: D_kwDOS1Ul_s4AnA6L
discussion_url: 'https://github.com/yuanchaodu/blog/discussions/4'
---

# Linux 安装指南

<img src="images/Linux.svg" width="300">

如果你是第一次安装 Linux，推荐从桌面版发行版开始，例如：Ubuntu、Linux Mint 或 Fedora。

## 一、安装前准备

### 1. 准备硬件

* 一台电脑
* 一个 8GB 以上的 U 盘
* 稳定的网络连接

### 2. 下载 Linux 镜像

前往发行版官网下载 ISO 镜像文件，例如：

* [Ubuntu 官网](https://ubuntu.com/download)
* [Linux Mint 官网](https://linuxmint.com/download.php)
* [Fedora 官网](https://fedoraproject.org/workstation/download)

### 3. 制作启动 U 盘

常用工具：

| 系统                  | 工具                                                              |
| ------------------- | --------------------------------------------------------------- |
| Windows             | [Rufus](https://rufus.ie)                |
| Windows/Linux/macOS | [balenaEtcher](https://etcher.balena.io) |

制作步骤：

1. 插入 U 盘
2. 打开工具
3. 选择 ISO 文件
4. 选择目标 U 盘
5. 点击开始写入

制作启动 U 盘最简单的方法:
> 直接解压下载好的 ISO 镜像文件，然后把镜像文件内容拷贝到 U 盘根目录（最简单的方法）

---

## 二、启动安装程序

### 1. 进入 BIOS/UEFI

开机时连续按：

* F2
* F10
* F12
* ESC
* DEL

具体按键因品牌而异。

### 2. 设置 U 盘启动

在 Boot 菜单中：

* 将 USB Device 调整为第一启动项
* 保存并重启

---

## 三、安装 Linux

启动后一般会看到：

* Try Linux（试用）
* Install Linux（安装）

选择安装即可。

### 1. 语言设置

选择：

* 中文（简体）
* English

### 2. 键盘布局

通常选择：

* Chinese
* US Keyboard

### 3. 网络配置

连接：

* 有线网络
* Wi-Fi

### 4. 安装类型

#### 方案一：全盘安装

适合：

* 不保留原系统
* 新电脑

安装程序自动完成分区。

#### 方案二：与 Windows 双系统

适合：

* 保留 Windows
* 学习 Linux

安装程序通常会显示：

> Install alongside Windows

直接选择即可。

#### 方案三：手动分区

适合有经验用户。

常见分区：

| 挂载点   | 建议容量      |
| ----- | --------- |
| /     | 30GB~50GB |
| /home | 剩余空间      |
| swap  | 4GB~16GB  |

---

## 四、创建用户

设置：

* 用户名
* 计算机名
* 登录密码

建议：

* 密码不少于 8 位
* 包含大小写字母和数字

---

## 五、完成安装

安装完成后：

1. 点击 Restart
2. 拔出 U 盘
3. 系统自动进入 Linux

---

## 六、安装后常用操作

### 更新系统

以 Ubuntu 为例：

```bash
sudo apt update
sudo apt upgrade -y
```

### 安装常用软件

浏览器：

```bash
sudo apt install firefox
```

编辑器：

```bash
sudo apt install vim
```

SSH：

```bash
sudo apt install openssh-server
```

查看系统信息：

```bash
uname -a
```

---

## 七、企业环境推荐

如果你从事服务器、信息化和工业企业 IT 工作，可以考虑：

| 场景     | 推荐系统           |
| ------ | -------------- |
| 办公桌面   | Ubuntu Desktop |
| 开发测试   | Fedora         |
| 服务器    | Ubuntu Server  |
| 企业级服务器 | Rocky Linux    |
| 数据中心   | Debian         |

对于化工企业数字化工厂、MES、数据库、中间件、虚拟化平台等场景，目前使用较多的是 Ubuntu Server、Rocky Linux 和 Debian。
