---
title: 命令行终端中使用代理下载
date: 2017-02-02T02:47:34+08:00
categories: ["MacOS"]
tags: [Linux]
---

摘要：你是否还在烦恼有时下载一个软件半天下载不了？要么超时要么连接不上，因为被墻，别头疼了，ProxyChains-NG能帮你解决烦恼，需要在命令行中下载一些需要fq才能下载的软件，这时就可以安装及配置 ProxyChains-NG 实现终端下代理，下载介绍如何安装及配置。

<!-- more -->
项目主页：<https://github.com/rofl0r/proxychains-ng>
### 一.下载安装ProxyChains-NG
    # Mac OS
    brew install proxychains-ng
    # Ubuntu
    sudo apt install proxychains-ng

### 二.配置
编辑配置文件`/usr/local/etc/proxychains.conf`[Mac OS] `/etc/proxychains.conf`[Ubuntu]
在 [ProxyList] 下面（也就是末尾）加入代理类型，代理地址和端口，可以选择本地`127.0.0.1`,也可以选择局域网内的其他代理服务器，并根据自己的代理类型进行配置。
例如使用 TOR 代理，注释掉原来的代理并添加

    socks5  127.0.0.1 1080  #socks5代理
    http    192.168.100.88 8123  #http代理

### 三.使用

    proxychains4 wget youtube.com

如果返回一堆html代码说明配置成功了。

如果使用有问题请参考：
1、<https://eliyar.biz/proxy-for-mac-terminal/>
