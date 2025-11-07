---
title: 更新CentOS7.4到CentOS7.5
date: 2018-03-12T02:47:34+08:00
categories: [CentOS]
tags: [CentOS]
---
2018年5月15日，CentOS官方发布了最新的CentOS7.5（1804）系统，加入了一些新的特性，下面介绍如何更新系统到最新的CentOS7.5。
<!--more-->

### 更新方法
#### 1、查看当前的系统版本
```sh
$ cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```

#### 2、更新系统
```sh
$ sudo yum clean all
$ sudo yum upgrade
$ sudo systemctl reboot
```

#### 3、查看新的系统版本
```sh
$ cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)
```

### 参考链接
1、[官方博客](https://blog.centos.org/2018/05/centos-7-5-1804-released/)
2、[下载地址](https://opsx.alibaba.com/mirror)

