---
title: 内网穿透服务Frp
date: 2018-01-31T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---

frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp, http, https 协议。当想从公网环境访问家里或公司局域网内的服务器时便可以使用frp搭建内网穿透服务实现此功能。

<!--more-->

### 1、原理介绍

Frp是`C/S`架构，在具有公网IP的服务器上搭建Frp的服务端，在本地局域网搭建Frp客户端；Frp通过将局域网内的IP端口映射到公网IP的某个端口上，当我们访问公网IP的这个端口时，相当于访问了其映射的局域网内的 IP:Port

### 2、服务条件

- 具有公网IP的服务器
- 客户端和服务端使用相同或相近的版本

### 3、服务端搭建

下载对应系统的最新的程序[frp](https://github.com/fatedier/frp/releases) 解压缩，进入解压目录
修改配置文件

```sh
$ vim frps.ini
[common]
bind_port = 7000
bind_udp_port = 7001
kcp_bind_port = 7000

dashboard_port = 7500
# dashboard 用户名密码，默认都为 admin
dashboard_user = admin
dashboard_pwd = admin
vhost_http_port = 80

token = xxxx #自定义
max_pool_count = 5
max_ports_per_client = 0
authentication_timeout = 900
subdomain_host = xxxxx.com
tcp_mux = true
```

启动Frp服务端

```sh
$ nohup ./frps -c ./frps.ini &
```

### 4、客户端搭建

下载相同的程序
修改配置文件

```sh
$ vim frpc.ini
[common]
server_addr = x.x.x.x   #公网IP地址
server_port = 7000

log_file = /usr/local/frp/frpc.log
# trace, debug, info, warn, error
log_level = info
log_max_days = 3

token = xxxxxx #修改
pool_count = 5
tcp_mux = true
user = pi
login_fail_exit = true
protocol = tcp

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[gogs]
type = http
local_ip = 172.16.10.80
local_port = 3000
use_encryption = true
use_compression = false
subdomain = gogs
#custom_domains = gogs.xxxx.com
```

启动本地客户端

```sh
$ nohup ./frpc -c ./frpc.ini &
```

ssh连接局域网内的服务器

```sh
$ ssh -oPort=6000 root@x.x.x.x
```

也可在局域网搭建能够被公网访问的Web服务器，开启80或443端口，更多高级的配置请参考下面的文档。

### 5.systemctl管理frpc服务

```sh
$ vim /lib/systemd/system/frpc.service
[Unit]
Description=frpc daemon
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/frp/frpc -c /usr/local/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

服务端同理，把`frpc`改为`frps`

### 5、参考文档

1、[Frp说明文档](https://github.com/fatedier/frp/blob/master/README_zh.md)
