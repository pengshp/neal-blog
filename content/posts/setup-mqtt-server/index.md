---
title: 使用mosquitto设置一个MQTT服务
date: 2019-05-16T02:47:34+08:00
categories: [IoT]
tags: [IoT]
---
![mqtt](https://hugoblog-img-1251694304.cos.ap-guangzhou.myqcloud.com/blog/mqtt-linux.png "mqtt")


MQTT 是一种基于 pusblish/subscribe(发布/订阅)的消息通信协议，适用于 M2M (machine to machine)在低带宽条件下进行消息通信。广泛运用于物联网(IoT)领域。本文介绍使用 mosquitto 搭建一个 MQTT Broker。

<!--more-->

### 1、下载安装mosquito

```sh
$ sudo yum install -y mosquitto
```

### 2、修改配置文件

```sh
$ sudo vim /etc/mosquitto/mosquitto.conf
pid_file /var/run/mosquitto.pid

persistence true
persistence_location /var/lib/mosquitto/

log_dest file /var/log/mosquitto/mosquitto.log

include_dir /etc/mosquitto/conf.d

bind_address 172.16.10.80 # 绑定的IP地址

allow_anonymous false # 禁止匿名登录
protocol mqtt
password_file /etc/mosquitto/pwfile  #密码文件
port 1883  # 监听的端口号

max_connections 100 # 最大连接数
```

### 3、设置密码

```sh
# 为用户mqtt设置一个密码
$ mosquitto_passwd -c /etc/mosquitto/pwfile mqtt
```

### 4、启动服务

```sh
$ sudo systemctl start mosquitto.service
$ sudo systemctl enable mosquitto.service
```

### 5、测试服务

#### 订阅端

```sh
$ mosquitto_sub -h 172.16.10.80 -d -u mqtt -P hello -t mqtt/test
```

#### 发布端

```sh
$ mosquitto_pub -h 172.16.10.80 -d -u mqtt -P hello -t mqtt/test -m "Hello!"
```

> -h    指定MQTT的IP
>
> -d    打开调试模式
>
> -u    指定用户名
>
> -P    指定密码
>
> -t     指定主题Topic
>
> -m   指定消息内容

一端从 MQTT Broker 订阅一个主题，另一端向 MQTT Broker 发布主题，发布后订阅端便可以收到发布端发布的消息。
