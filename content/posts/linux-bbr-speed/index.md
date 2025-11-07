---
title: Linux VPS 开启 bbr 加速和fastopen
date: 2020-07-16T02:47:34+08:00
categories: [Linux]
tags: [Linux,VPS]
---



![blog-linux-bbr](https://pengshp.coding.net/p/images/d/images/git/raw/master/blog-linux-bbr.png "Linux BBR")

TCP BBR 是由 Google 设计，于2016年发布的拥塞算法，可以降低 TCP 握手的延迟时间，加快 VPS 代理的速度，本文介绍如何在 VPS上开启 bbr,并打开`tcp_fastopen`

<!--more-->



## 1.bbr简介

TCP BBR（Bottleneck Bandwidth and Round-trip propagation time）是由Google设计，于2016年发布的拥塞算法。以往大部分拥塞算法是基于丢包来作为降低传输速率的信号，而BBR则基于模型主动探测。该算法使用网络最近出站数据分组当时的最大带宽和往返时间来创建网络的显式模型。数据包传输的每个累积或选择性确认用于生成记录在数据包传输过程和确认返回期间的时间内所传送数据量的采样率。该算法认为随着网络接口控制器逐渐进入千兆速度时，分组丢失不应该被认为是识别拥塞的主要决定因素，所以基于模型的拥塞控制算法能有更高的吞吐量和更低的延迟，可以用BBR来替代其他流行的拥塞算法，例如CUBIC。Google在YouTube上应用该算法，将全球平均的YouTube网络吞吐量提高了4%，在一些国家超过了14%。

BBR之后移植入Linux内核4.9版本，并且对于QUIC可用。

BBR 目的是要尽量跑满带宽，并且尽量不要有排队的情况，效果并不比速锐差。

Linux kernel 4.9+ 已支持 tcp_bbr。下面简单讲述基于 KVM 架构 VPS 如何开启。

## 2.查看Linux内核版本

```shell
[xdl@vultr ~]$ uname -r
5.2.2-1.el7.elrepo.x86_64
```



## 3.开启bbr加速

```shell
[xdl@vultr ~]$ sudo vim /etc/sysctl.d/bbr.conf
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
```

## 4.重新加载配置

```shell
[xdl@vultr ~]$ sudo sysctl -p
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
```



## 5.测试是否生效

```shell
[xdl@vultr ~]$ lsmod | grep bbr
tcp_bbr                20480  246
```



## 6.检查tcp_fastopen

TCP fast open 集成到了Linux kernel3.7,并且在3.13的内核中默认开启，使用`uname -r`可查看当前系统的内核版本。或者使用下面的命令检查

```shell
[xdl@vultr ~]$ cat /proc/sys/net/ipv4/tcp_fastopen
1
```

> It can return 4 values.
>
> - 0 means disabled.
> - 1 means it’s enabled for outgoing connection (as a client).
> - 2 means it’s enabled for incoming connection (as a server).
> - 3 means it’s enabled for both outgoing and incoming connection.



## 7.开启fast open

由于我们的VPS 是作为server 使用，可以设置为3，对流入和流出的流量都生效。在LAN客户端可以使用默认1便可。

```shell
[xdl@vultr ~]$ sudo vim /etc/sysctl.d/bbr.conf
net.ipv4.tcp_fastopen=3

# 重载配置
[xdl@vultr ~]$ sudo sysctl -f /etc/sysctl.d/bbr.conf
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
net.ipv4.tcp_fastopen=3
```

如果系统内核低的可以使用下面链接的一键开启bbr加速脚本，也会安装最新你的内核。支持CentOS6+，Debian8+，Ubuntu16+

参考链接

1. [How to Install Shadowsocks-Libev Proxy Server on Debian 10 VPS](https://www.linuxbabe.com/debian/install-shadowsocks-libev-proxy-server-debian-10)

2. [一键安装最新内核并开启 BBR 脚本](https://teddysun.com/489.html)
