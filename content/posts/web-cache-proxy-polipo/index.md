---
title: Web 缓存代理服务器polipo
date: 2018-07-02T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---
Polipo 是一个小而快速的缓存web 代理程序(web 缓存, HTTP 代理, 代理服务器)。可以实现HTTP和SOCKS代理。为了最小化延迟，Polipo管线化多个资源请求，在同一个TCP/IP连接上多路复用。Polipo具有HTTP 1.1兼容，支持IPv4、IPv6，流量过滤和隐私增强。
<!--more-->

测试环境：CentOS7

### 下载编译安装
```sh
# 下载源码
$ wget https://github.com/jech/polipo/archive/polipo-1.1.1.tar.gz

# 解压
$ tar -zxf polipo-1.1.1.tar.gz

# 安装依赖
$ yum install texinfo gcc make

# 编译
$ cd polipo-polipo-1.1.1/
$ make all

# 安装
$ su -c 'make install'
```

### 创建相关的配置文件
```sh
# 创建日志文件
$ mkdir /var/log/polipo
$ touch /var/log/polipo/polipo.log

# 创建配置目录
$ mkdir /etc/polipo
```


### 新建服务启动文件

```sh
$ vim /usr/lib/systemd/system/polipo.service
[Unit]
Description=polipo web proxy
After=network.target

[Service]
Type=simple
WorkingDirectory=/tmp
User=root
Group=root
ExecStart=/usr/local/bin/polipo -c /etc/polipo/config
Restart=always
SyslogIdentifier=Polipo

[Install]
WantedBy=multi-user.target
```

### 创建服务配置文件
```sh
$ vim /etc/polipo/config
logSyslog = true
socksParentProxy = "localhost:1080" #上层Socks5代理
socksProxyType = socks5
logFile = /var/log/polipo/polipo.log
logLevel = 4
proxyAddress = "0.0.0.0" #监听IP
proxyPort = 8123
allowedClients = 192.168.10.0/24 #允许连接的客户端
chunkHighMark = 50331648 #使用的较多的内存
objectHighMark = 16384
diskCacheRoot = "/var/cache/polipo" #缓存文件存储目录
cacheIsShared = true #分享缓存
serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
dnsQueryIPv6 = no #不适用IPv6，加快DNS查询速度
dnsUseGethostbyname = yes #使用系统的resolver
```

### 启动代理服务
```shell
$ systemctl start polipo.service
$ systemctl enable polipo.service
```

### 基本使用
把浏览器的代理配置为`polipo`服务器的IP地址，端口为8123，便可以使用`polipo`代理上网，加快网页的访问速度。

### 说明：

如果实在树莓派上使用，可直接使用apt安装，并复制配置文件直接修改配置,可直接使用`systemctl`管理服务。

```shell
$ cp /usr/share/doc/polipo/examples/config.sample /etc/polipo/config
```



