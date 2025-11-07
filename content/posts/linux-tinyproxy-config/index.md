---
title: Linux配置Tinyproxy代理
date: 2020-08-20T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---



`Tinyproxy`是一个轻量级的http代理软件，速度快体积小，可以使用来把socks5代理转为http代理。测试环境为 :fire: ArchLinux,IP:172.16.10.18,Socks5端口1090.

<!--more-->

## 查询软件包

```shell
~$ pacman -Ss tinyproxy
community/tinyproxy 1.10.0-2 [已安装]
    A light-weight HTTP proxy daemon for POSIX operating systems.
```

## 安装tinyproxy

```shell
~$ sudo pacman -Sy tinyproxy
```

## 修改配置文件

```bash
~$ vim /etc/tinyproxy/tinyproxy.conf
User tinyproxy
Group tinyproxy

Port 8888

Listen 0.0.0.0

Timeout 600

ErrorFile 404 "/usr/share/tinyproxy/404.html"
ErrorFile 400 "/usr/share/tinyproxy/400.html"
ErrorFile 503 "/usr/share/tinyproxy/503.html"
ErrorFile 403 "/usr/share/tinyproxy/403.html"

DefaultErrorFile "/usr/share/tinyproxy/default.html"


StatFile "/usr/share/tinyproxy/stats.html"

LogFile "/var/log/tinyproxy/tinyproxy.log"

#关闭syslog，使用自定义的日志文件
#Syslog On

LogLevel Info

PidFile "/var/run/tinyproxy/tinyproxy.pid"

# 在请求头中加入客户端IP
#XTinyproxy Yes

#设置上游代理，转换上游socks5代理为http代理
upstream socks5 127.0.0.1:1090

MaxClients 100

MinSpareServers 5
MaxSpareServers 20

StartServers 10

MaxRequestsPerChild 0

# 允许局域网的哪些主机访问
Allow 172.16.10.0/24
Allow 192.168.1.0/24

AddHeader "X-My-Header" "Powered by Tinyproxy"
ViaProxyName "tinyproxy"
```

## 启动服务

```shell
~$ sudo systemctl start tinyproxy.service
~$ sudo systemctl enable tinyproxy.service
```

## 测试

在局域网内的其它主机上把代理设置为上面的Tinyproxy,测试请求头信息

```shell
xdl@aml:~$ export http_proxy='http://172.16.10.18:8888'
xdl@aml:~$ export https_proxy='http://172.16.10.18:8888'

xdl@aml:~$ curl -I www.baidu.com
HTTP/1.0 200 OK
Via: 1.1 tinyproxy (tinyproxy/1.10.0)   # <------ 请求头中包含了Tinyproxy
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Content-Type: text/html
Etag: "575e1f6f-115"
Last-Modified: Mon, 13 Jun 2016 02:50:23 GMT
Pragma: no-cache
Server: bfe/1.0.8.18
Accept-Ranges: bytes
Date: Thu, 20 Aug 2020 06:57:01 GMT
Content-Length: 277
```



## 参考：

1.<https://tinyproxy.github.io/>

2.<https://github.com/tinyproxy/tinyproxy>

3.<https://linux.cn/article-7119-1.html>

