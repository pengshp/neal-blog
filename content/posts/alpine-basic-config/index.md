---
title: Alpine 基本配置
date: 2018-12-02T02:47:34+08:00
categories: [linux]
tags: [linux]
---

alpine作为一个轻量级docker容器已经是很流行了，alpine linux也是一个很轻量级的linux系统，网上关于docker下配置alpine的例子不少，很少有基于alpine下配置docker环境的，本例即为虚拟机下安装alpine同时配置docker环境

<!--more-->

Alpine 操作系统是一个面向安全的轻型 Linux 发行版。目前 Docker 官方已开始推荐使用 Alpine 替代之前的 Ubuntu 做为基础镜像环境。这样会带来多个好处。包括镜像下载速度加快，镜像安全性提高，主机之间的切换更方便，占用更少磁盘空间等。

## 下载alpine linux镜像

http://mirrors.aliyun.com/

## 配置虚拟机

资源分配根据需要可调整，安装过程需要连接外网，dhcp分配IP，开启虚拟机，使用root登录系统，此时不需要密码，直接登录。

## 安装alpine linux

执行setup-alpine进行安装设置，设置完后重启进入系统

## 安装后配置

使用root登录，先新进一个用户，alpine linux默认不能使用root远程登录

    useradd tem
    passwd tem

使用tem 用户远程登录后在切换到root
su - root

## 修改镜像源

```bash
$ vi /etc/apk/repositories
http://mirrors.ustc.edu.cn/alpine/v3.7/main
http://mirrors.ustc.edu.cn/alpine/v3.7/community
```

## 基本使用

```sh
apk update
apk add vim git

# 搜索package
$ apk search vim

# 删除package
$ apk del vim

# 查看package信息
$ apk info vim
```

## 安装docker

```
apk add docker
rc-update add docker boot    #加入开机自启
service docker start
docker version               #查看版本信息
```

## 安装docker-compose

```sh
apk add py-pip
pip install docker-compose
```

## 软件包在线查询

https://pkgs.alpinelinux.org/packages
