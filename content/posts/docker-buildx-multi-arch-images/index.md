---
title: 使用 Docker buildx 构建多架构镜像
date: 2020-07-16T02:47:34+08:00
categories: [Docker]
tags: [Docker, Linux]
---

![blog-docker-buildx](https://pengshp.coding.net/p/images/d/images/git/raw/master/blog-docker-buildx.png "buildx")

Docker Buildx 是一个 docker CLI 插件，其扩展了 docker 命令，支持 Moby BuildKit 提供的功能。提供了与 docker build 相同的用户体验，并增加了许多新功能。

<!--more-->

## 1.简介

BuildKit 是下一代的镜像构建组件，主要特点有很多，本文主要使用其可以编译多种系统架构的特性。
网址：https://github.com/moby/buildkit
需要注意的是，该功能仅适用于 Docker v19.03+ 版本。

本文将讲解如何使用 Buildx 构建多种系统架构的镜像。
在开始之前，已经默认你在 Linux 系统（各大发行版）下安装好了 64 位的 Docker，我的测试环境为Linux amd64`Ubuntu20.04`.

## 2.开始实验性功能

```sh
~$ vim /etc/docker/daemon.json
{
    "experimental": true
}

# 重启docker使配置生效
~$ sudo systemctl restart docker.service

~$ docker version
Server: Docker Engine - Community
 Engine:
  Version:          20.10.1
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       f001486
  Built:            Tue Dec 15 04:35:42 2020
  OS/Arch:          linux/amd64
  Experimental:     true   # <--------开启成功
```

## 2.安装跨平台编译环境依赖支持

```sh
~$ apt install qemu binfmt-support qemu-user-static
```

## 3.安装buildx

在最新的Docker 20.10.1+ 版本中已默认安装buildx，低于这个版本则手动安装

下载地址：<https://github.com/docker/buildx/releases/>

```sh
xdl@ubuntu20:~$ mkdir -p ~/.docker/cli-plugins
xdl@ubuntu20:~$ mv buildx-v0.4.1.linux-amd64 .docker/cli-plugins/docker-buildx
xdl@ubuntu20:~$ chmod a+x ~/.docker/cli-plugins/docker-buildx
```

## 4.准备环境

```sh
xdl@ubuntu20:~$ docker buildx ls
NAME/NODE DRIVER/ENDPOINT STATUS  PLATFORMS
default * docker
  default default         running linux/amd64, linux/386
xdl@ubuntu20:~$ docker buildx create --name mybuilder \
			--platform linux/arm,linux/arm64,linux/amd64
mybuilder

xdl@ubuntu20:~$ docker buildx use mybuilder
xdl@ubuntu20:~$ docker buildx inspect
Name:   mybuilder
Driver: docker-container

Nodes:
Name:      mybuilder0
Endpoint:  unix:///var/run/docker.sock
Status:    inactive   # <------- 未运行
Platforms: linux/arm/v7*, linux/arm64*, linux/amd64*

$ docker buildx inspect --bootstrap
[+] Building 3.5s (1/1) FINISHED
 => [internal] booting buildkit                           3.5s
 => => pulling image moby/buildkit:buildx-stable-1        3.2s
 => => creating container buildx_buildkit_mybuilder0      0.4s
Name:   mybuilder
Driver: docker-container

Nodes:
Name:      mybuilder0
Endpoint:  unix:///var/run/docker.sock
Status:    running   # <-------- 运行状态
Platforms: linux/arm/v7*, linux/arm64*, linux/amd64*, linux/386
```

## 5.登录Docker Hub

```sh
# 登录 DockerHub 便于push images
~$ docker login
```

## 6.编译示例

下面使用`Trojan`源码编译适用多CPU架构的 docker 镜像，系统配置越高编译越开，需要设置科学上网环境或者在国外的VPS上编译。由于我常使用的CPU架构有 `armv7,arm64,amd64`,其它的可自己添加，我就只编译这三种架构的镜像。

```sh
# 下载最新Trojan源码
$ wget https://github.com/trojan-gfw/trojan/archive/v1.16.0.zip
$ unzip *.zip
$ cd trojan

# 加入自定义的设置
$ vim Dockerfile

$ docker buildx build --platform linux/arm,linux/arm64,linux/amd64 \
                      -t pengshp/trojan:v1.16 . --push
```

> --platform 指定要编译的CPU架构
>
> --push 编译完后上传到 Docker Hub

![image-20200716162421820](https://pengshp.coding.net/p/images/d/images/git/raw/master/image-20200716162421820.png "Docker Hub")

### 参考链接

1.[使用 Docker Buildx 构建多种系统架构镜像](https://teddysun.com/581.html)

2.[Github buildx](https://github.com/docker/buildx/)

3.[Docker buildx](https://docs.docker.com/buildx/working-with-buildx/)
