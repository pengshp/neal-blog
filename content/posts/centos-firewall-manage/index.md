---
title: CentOS 防火墙 firewalld 的管理
date: 2019-11-02T02:47:34+08:00
categories: [CentOS]
tags: [CentOS]
---

CentOS7.x 使用firewalld 防火墙替代了 CentOS6.x 中的 iptables,两者的使用上有较大的区别，本文介绍firewall在日常的使用和管理中常使用的一些基本命令。

<!--more-->

### 一、firewall基本管理

```bash
# 开启防火墙服务
$ sudo systemctl start firewalld.service

# 关闭防火墙
$ sudo systemctl stop firewalld.service

# 开机自启
$ sudo systemctl enable firewalld.service

# 关闭开机自启
$ sudo systemctl disable firewalld.service
```

### 二、firewall-cmd

#### 1、查看状态

```bash
$ firewall-cmd --state
# running表示运行
```

#### 2、获取活动的区域

```bash
$ firewall-cmd --get-active-zones
```

#### 3、获取所有支持的服务

```bash
$ firewall-cmd --get-service
```

#### 4、开放某个服务

```bash
# 临时，重启后失效
$ firewall-cmd --zone=public --add-service=https

# 永久
$ firewall-cmd --permanent --zone=public --add-service=https

# 重载防火墙使生效
$ firewall-cmd --reload
```

#### 5、开放某个端口

```bash
# 临时
$ firewall-cmd --zone=public --add-port=8080/tcp

# 永久
$ firewall-cmd --permanent --zone=public --add-port=8080/tcp

# 重载防火墙使生效
$ firewall-cmd --reload
```

#### 6、查看开启的服务和端口

```sh
# 开启的服务
$ firewall-cmd --permanent --zone=public --list-services

# 开启的端口
$ firewall-cmd --permanent --zone=public --list-ports
```

#### 7、查询服务的启用状态

```bash
$ firewall-cmd ---query-service http
```

#### 8. 查看当前`zone`中的规则

```sh
$ firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

#### 9. 默认`zone`管理

```sh
$ firewall-cmd --get-default-zone
public

# 设置默认的zone
$ firewall-cmd --set-default-zone=work
$ firewall-cmd --set-default-zone=public
```

10. #### 删除规则

```sh
[root@localhost ~]$ firewall-cmd  xxx
--remove-service=http ##在home区域内将http服务删除在开放列表中删除
--remove-port=8080 # s
--remove-interface=eth0 ##将网络接口在默认的区域内删除
--remove-source=192.168.1.1 ##删除源地址的流量到指定区域
```

常用的命令如上所示，基本能够满足日常管理和使用。
