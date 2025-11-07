---
title: CentOS搭建FTP服务器
date: 2018-04-02T02:47:34+08:00
categories: [CentOS]
tags: [Linux, CentOS]
---

FTP是文件传输协议，可实现Windows和Linux之间快速的上传和下载文件。CentOS下搭建FTP服务器一般使用`vsftpd`,搭建的环境为`CentOS7.4`,下面是搭建和配置过程。

<!--more-->

vsftpd（Very Secure FTP Daemon）是一个在类 Unix 系统（如 Linux）上运行的开源 FTP 服务器软件。其名称中的 "Very Secure" 明确表达了它的设计目标：安全、高效、稳定。这也是它成为 Linux 发行版中最常用 FTP 服务的原因之一（如 CentOS、Ubuntu、Red Hat 等默认推荐或预装）。

### 安装FTP服务

```sh
$ sudo yum -y install vsftpd
```

### 配置FTP服务器

#### 了解vsftpd配置

vsftpd的配置目录为`/etc/vsftpd`,主要包括下面的配置文件：

- vsftpd.conf 主配置文件
- ftpusers 配置禁止访问FTP服务器的用户列表
- user_list 配置用户访问控制

FTP服务器的身份验证模式：

- 匿名开放---无需密码验证可登陆FTP服务器，最不安全
- 本地用户---通过Linux本地的账号密码进行验证，存在暴力破解的风险，相对安全；
- 虚拟用户---FTP传输服务单独建立数据库文件，虚拟出来口令验证的账户信息，但账号是在服务器系统不存在，仅供FTP传输服务验证使用，最安全；
  为了保证安全性，本文配置的FTP服务器为匿名访问模式；

#### 阻止匿名访问和切换根目录

匿名访问和切换根目录都会给服务器带来安全风险，建议关闭这两个功能。编辑主配置文件

```sh
$ sudo vim /etc/vsftpd/vsftpd.conf
 # 禁止匿名用户
anonymous_enble=NO
 # 禁止切换根目录
chroot_local_user=YES
 # 开启虚拟用户模式
guest_enable=YES
listen=YES
listen_port=21
download_enable=YES
```

编辑完后`:wq`保存退出。

#### 创建FTP用户

创建一个新的用户`ftpuser`

```sh
$ sudo useradd ftpuser
```

为用户ftpuser设置密码，xxx改为自己的密码

```sh
$ echo "xxxxx" |passwd ftpuser --stdin
```

#### 限制FTP用户只能FTP访问

为了安全，限制用户ftpuser只能通过FTP访问服务器，而不能直接登录服务器。

```sh
$ usermod -s /sbin/nologin ftpuser
```

#### 为用户分配主目录

FTP服务器默认的访问目录为`/var/ftp`

### 启动FTP服务器并开机自启

```sh
$ sudo systemctl start vsftpd
$ sudo systemctl enble vsftpd
```

### 设置防火墙允许FTP访问

设置防火墙永久允许FTP访问

```sh
$ firewall-cmd --add-service=ftp --permanent
$ firewall-cmd --reload  # 重载防火墙
```

### 访问FTP服务器

- 方案一：
  Windows的资源管理器地址栏里输入<ftp://ftpuser:xxxxx@192.168.22.156>

- 方案二：
  通过FTP客户端工具访问
  使用开源免费并且跨平台的`FileZilla`客户端访问，填入FTP服务器的IP地址，用户名和密码便可以访问。
