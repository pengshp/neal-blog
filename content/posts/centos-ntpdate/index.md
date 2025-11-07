---
title: Linux 的时间同步那些事
date: 2018-06-02T02:47:34+08:00
lastmod: 2021-12-04T20:58:26+08:00
description: "Linux时间同步"
categories: [CentOS]
tags: [CentOS]
---

Linux中很多服务需要进行时间同步，不然容易导致出错。本文介绍使用阿里云的时间服务器同步服务器的时间。下面介绍三种时间同步的方案。

<!--more-->

## 方案1：使用NTP同步时间

### 1、安装软件

```sh
$ yum install -y ntpdate
```

### 2、时间同步服务器

可用的时间同步服务器，阿里提供了一些NTP时间服务器可以用于从互联网中同步服务器的时间；`ntp.aliyun.com`

- ntp1.aliyun.com
- ntp2.aliyun.com
- ntp3.aliyun.com
- ntp4.aliyun.com

### 3、同步时间

从上面的时间同步服务器中选择一个进行时间同步。

```sh
$ ntpdate -u ntp.aliyun.com
```

### 4、加入计划任务

可以把时间同步加入系统的计划任务，定时从互联网同步时间

```sh
$ crontab -e
* */2 * * * ntpdate -u ntp1.aliyun.com &> /dev/null
# 每两个小时同步一次时间
$ systemctl start crond
$ systemctl enable crond
```

复杂，不便利，暂时不推荐使用。

## 方案2：使用chrony同步时间

### 1. 修改配置文件

```sh
[xdl@CentOS] ~$ sudo vim /etc/chrony.conf
pool ntp1.aliyun.com iburst
```

### 2. 启动服务

```sh
[xdl@CentOS] ~$ sudo systemctl enable --now chronyd.service
```

### 3. 查看状态

```sh
[xdl@CentOS] ~$ chronyc sources -v
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 120.25.115.20                 2   6    37    64   -285us[-4209us] +/-   12ms
```

暂时推荐使用 `chrony`进行时间同步。

## 方案3：使用systemd管理时间同步

systemd的组件中有一个`timesyncd`的服务专门用来管理时间同步。

### 安装软件

```sh
$ dnf install -y  systemd-timesyncd
```

### 修改配置文件

```sh
$ sudo vim /etc/systemd/timesyncd.conf
# See timesyncd.conf(5) for details.

[Time]
NTP=ntp.aliyun.com
```

### 启动服务

```sh
$ sudo systemctl stop chronyd.service
$ sudo systemctl enable --now systemd-timesyncd.service
$ sudo systemctl status systemd-timesyncd.service
```

### 验证时间同步状态

```sh
$ timedatectl status
               Local time: 六 2021-12-25 17:36:02 CST
           Universal time: 六 2021-12-25 09:36:02 UTC
                 RTC time: 六 2021-12-25 09:36:02
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes     # <----- 时间同步开启
              NTP service: inactive
          RTC in local TZ: no
```

## 总结：

随着`systemd`的普及，更推荐使用方案3，简单，和系统结合性更好，而且占用的内存更小。
