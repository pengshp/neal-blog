---
title: 使用 OpenSSL 自签名 TLS/SSL 证书
date: 2020-07-15T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---

有时需要一个本地的自签名证书，便于本地的加密https 访问，或者用于开发和测试，OpenSSL 便可以方便自签名一个证书，下面介绍使用方法。

<!--more-->

### 证书的工作原理

你使用自己的证书请求文件 `server.csr`向证书签发机构CA如：Verisign、GoDaddy 请求证书，证书签发机构使用它们的root根证书和私钥根据你提供的证书请求文件签发你需要的证书`server.crt`,所有的浏览器或者操作系统中都有一份证书签发机构的root根证书拷贝，当访问你的证书签发的域名时，会使用签发机构的root证书验证你的域名对应证书的有效性。

自签名TLS/SSL证书过程：OpenSSL 生成一个私钥server.key，使用私钥生成证书请求文件server.csr,再使用server.key 和 server.csr 生成证书文件server.crt

## 1、安装OpenSSL

```shell
# Debian
~$ sudo apt install -y openssl

# CentOS
~$ sudo yum install -y openssl

# 查看版本
~$ openssl version
OpenSSL 1.1.1g  21 Apr 2020
```

## 2、生成CSR证书请求文件和私钥

使用秋水逸冰大佬制作的网站在线生成,点击  <https://www.csr.sh/>

![image-csr](https://pengshp.coding.net/p/images/d/images/git/raw/master/image-20200715143203262.png "csr")

域名可以填写泛域名，这样可以匹配`www.alarmpi.lan`,`dl.alarmpi.lan`

点击生成CSR，复制出来用编辑器打开，分两部分，上半部分为证书的请求文件，下半部分为私钥，分别保存为`server.csr`,`server.key`

```shell
openssl$ ls
server.csr  server.key
```



## 3、使用OpenSSL生成服务证书

```shell
~$ openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
Signature ok
subject=C = CN, CN = *.alarmpi.lan, L = Guangzhou, O = RaspberryPi, ST = Guangdong, OU = IT
Getting Private key

~$ $ ls
server.crt  server.csr  server.key
```

`-days`指定证书的有效期，这个证书的有效期为10年。

## 4、验证证书的有效性

```shell
~$ openssl verify server.crt
```

如果要你的浏览器或操作系统信任你的自签名证书，即访问时不再有警告提示，需要你把用于签发证书的root根证书导入到浏览器或者操作系统的`受信任的证书签发机构`域。

### 参考链接

1、[How to create & sign SSL/TLS certificates](https://dev.to/techschoolguru/how-to-create-sign-ssl-tls-certificates-2aai)

2、[OpenSSL command cheatsheet](https://www.freecodecamp.org/news/openssl-command-cheatsheet-b441be1e8c4a/)

3、[生成证书的命令行工具](https://github.com/FiloSottile/mkcert)
