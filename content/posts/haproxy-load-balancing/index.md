---
title: HAProxy实现负载均衡
date: 2018-02-01T02:47:34+08:00
categories: [CentOS]
tags: [CentOS]
---

当随着访问量的增加时，原来的单台服务器难以承受大量的访问时，就需要使用负载均衡来分担服务器的压力。本文介绍使用`HAProxy`实现一个免费、高效、可靠的高可用负载均衡解决方案。

<!--more-->

### HAProxy简介

HAProxy（High Availability Proxy）是一款**高性能的TCP/HTTP负载均衡器和反向代理服务器**，由法国人Willy Tarreau开发，采用C语言编写。它以**轻量、高效、稳定**著称，广泛应用于高并发场景，如大型网站、API服务、数据库负载均衡等，是构建高可用架构的核心组件之一。适合处理高负载站点的七层数据请求，对外可屏蔽内部的真实Web服务器，防止内部服务器遭受外部攻击。

### **核心功能**

1. **负载均衡**
   - **支持多种协议**：HTTP、HTTPS、TCP（如数据库、SSH）、UDP（v2.3+实验性支持）。
   - **丰富的负载均衡算法**：
     - 静态算法：轮询（Round Robin）、加权轮询（Weighted RR）、源IP哈希（Source IP Hash，会话保持）。
     - 动态算法：最小连接数（Least Connections）、加权最小连接（Weighted LC）、URL哈希（URL Hash）等。
   - **健康检查**：主动/被动检查后端服务器状态，自动剔除故障节点，恢复后重新加入。

2. **反向代理**
   - 隐藏后端服务器，保护内部架构，统一入口流量。
   - 支持路径路由（基于URL路径转发到不同后端服务，如`/api`→API服务器，`/static`→静态资源服务器）。
   - SSL/TLS终结：解密HTTPS流量，减轻后端服务器负担，同时支持SSL证书管理。

3. **高可用与性能优化**
   - **无状态设计**：支持水平扩展，可部署多实例配合Keepalived实现故障转移。
   - **低资源消耗**：单进程事件驱动模型（类似Nginx），处理十万级并发连接，内存占用极低。
   - **连接复用**：复用后端服务器TCP连接，减少握手开销。
   - **缓存与压缩**：缓存静态资源，支持Gzip压缩，提升响应速度。

4. **安全防护**
   - DDoS防护：限制单IP连接数、请求速率。
   - WAF基础功能：过滤恶意请求（如SQL注入、XSS）。
   - 访问控制：基于IP、用户认证（Basic Auth）等限制访问。

5. **监控与日志**
   - 内置统计页面（`/haproxy?stats`），实时展示流量、后端状态、错误率等指标。
   - 详细日志记录，支持自定义日志格式，便于问题排查和性能分析。

### **应用场景**

- **Web服务负载均衡**：分发HTTP/HTTPS请求到多台应用服务器（如Nginx、Tomcat、Node.js）。
- **数据库读写分离**：通过TCP模式代理MySQL，将读请求分发到从库，写请求路由到主库。
- **API网关**：统一API入口，实现路由、认证、限流、监控。
- **高可用集群**：配合Keepalived实现负载均衡器自身的故障转移，避免单点故障。
- **CDN节点流量分发**：在CDN架构中分发用户请求到边缘节点。

### **与其他工具对比**

| 工具        | 特点对比                                                                                            |
| ----------- | --------------------------------------------------------------------------------------------------- |
| **HAProxy** | 性能极强，支持TCP/UDP，配置灵活，适合复杂负载均衡场景，但HTTP功能（如缓存）较弱。                   |
| **Nginx**   | HTTP功能丰富（缓存、Rewrite、SSL），轻量易用，适合静态资源+反向代理，但TCP负载均衡能力弱于HAProxy。 |
| **LVS**     | 工作在Linux内核层（L4/L7），性能顶尖，但配置复杂，依赖内核模块，灵活性较低。                        |

### HAProxy解决方案

- 客户端IP：将客户端IP进行Hash计算并保存，当相同的IP访问代理服务器时可以转发至相同的后端服务器；
- Cookie：依靠真实服务器发送给客户端的Cookie信息进行会话保持；
- Session：保存真实服务器的Session及服务标识，实现会话保持功能；

### 实验拓扑图

![blog-haproxy2](https://pengshp.coding.net/p/images/d/images/git/raw/master/blog-haproxy2.png "Haproxy")

### 实验配置

#### HAProxy配置

1、安装HAProxy

```sh
$ yum install -y haproxy
```

2、修改配置文件

```sh
$ [root@CentOS7] ~$ vim /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

frontend haproxy_inbound
    bind *:80
    default_backend haproxy_httpd

backend haproxy_httpd
    balance     roundrobin
    option httpchk GET /index.html  # 健康检查
    server web1 192.168.10.129:80 check
    server web2 192.168.10.130:80 check
```

3、内核优化

```sh
$ vim /etc/security/limits.conf
*               soft    nofile        65535
*               hard    nofile        65535
```

4、开启日志服务

```sh
# 修改日志配置文件
$ vim /etc/rsyslog.conf
$ModLoad imudp
$UDPServerRun 514
local3.*                           /var/log/haproxy.log

# 重启日志服务
$ systemctl restart syslog
```

5、开启HAProxy

```sh
$ haproxy -f /etc/haproxy/haproxy.cfg
```

#### 后端Web服务器配置

后端服务器配置基本相同，为了验证试验效果，后端网页不同
1、配置网卡

```sh
$ vim /etc/sysconfig/network-scripts/ifcfg-ens33554984
NAME=ens33554984
BOOTPROTO='static'
ONBOOT="yes"
IPADDR=192.168.10.129
GATEWAY=192.168.10.128  #网关指向HAProxy的内网IP
TYPE="Ethernet"

$ systemctl restart network
```

2、Apache配置
网页文件做修改，其他配置相同

```sh
[root@CentOS7] ~$ yum install -y httpd

# Web-1
[root@CentOS7] ~$ echo "This is Web-1" > /var/www/html/index.html

# Web-2
[root@CentOS7] ~$ echo "This is Web-2" > /var/www/html/index.html

[root@CentOS7] ~$ systemctl start httpd
[root@CentOS7] ~$ systemctl enable httpd
```

#### 测试

重启HAProxy

```sh
$ systemctl restart haproxy.service
```

浏览器打开 <http://192.168.10.128>,刷新网页，便可以看到在`Web-1`和`Web-2`之间轮询。实现一个负载均衡。
