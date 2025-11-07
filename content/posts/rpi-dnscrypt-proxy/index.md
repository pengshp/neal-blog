---
title: 树莓派配置 dnscrypt-proxy
date: 2018-02-18T02:47:34+08:00
categories: [RaspberryPi]
tags: [RaspberryPi]
---
![dnscrypt](/images/dnscrypt.png "Dnscrypt")
`dnscrypt-proxy`可防止DNS污染，最新的2.0.x版本增加缓存DNS功能，加快DNS解析速度。
<!--more-->
### 下载
```shell
$ wget https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.44/dnscrypt-proxy-linux_arm-2.0.44.tar.gz
```

### 解压
```shell
$ tar -zxf dnscrypt-proxy-linux_arm-2.0.44.tar.gz -C /usr/local/
$ cd /usr/local/
$ mv linux-arm dnscrypt
```

### 修改配置文件
```sh
$ mv example-dnscrypt-proxy.toml dnscrypt-proxy.toml
$ vim dnscrypt-proxy.toml
server_names = ['rubyfish-ea', 'alidns-doh', 'cloudflare']
listen_addresses = ['127.0.0.1:53', '[172.16.10.10]:53']
log_file = 'dnscrypt-proxy.log'
fallback_resolver = '208.67.222.222:53'
cache = true
cache_size = 15000
...
```

把上面的`172.16.10.10`改为树莓派局域网内的IP地址。`server_names`会自动选择一个最快的使用，我的测试了`alidns-doh`[阿里服务器]延迟较低。`极客DNS`和`红鱼DNS`是中国大陆的服务器，其它一些高级的功能可根据自己的需要更改。

eg.

* 自定义域名解析
* 域名黑名单
* 过滤广告

### 安装服务
```shell
$ sudo /usr/local/dnscrypt/dnscrypt-proxy -service install
```

### 启动服务
```shell
$ sudo dnscrypt-proxy -service start
#  使用systemctl管理服务
$ sudo systemctl start dnscrypt-proxy.service
```

### 基本使用
```shell
$ sudo dnscrypt-proxy -service stop
$ sudo dnscrypt-proxy -service restart
$ sudo dnscrypt-proxy -service uninstall
```

### 使用方法
网络配置里修改DNS配置为树莓派的IP地址。便可以使用本地的DNS解析。

### 参考
[dnscrypt wiki](https://github.com/jedisct1/dnscrypt-proxy/wiki)
[dnscrypt-proxy](https://github.com/DNSCrypt/dnscrypt-proxy.git)
