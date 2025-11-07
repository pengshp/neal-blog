---
title: Ubuntu 编译OpenWrt与SDK
date: 2018-02-02T02:47:34+08:00
categories: [OpenWrt]
tags: [OpenWrt, sdk]
---

本文介绍如何使用ubuntu编译OpenWrt的SDK。

<!--more-->

## 一.开发准备

1.Ubuntu18.04 64bit
2.VMware
3.ar71xx架构路由器

## 二.开发环境搭建

1.安装依赖环境

```bash
sudo apt-get update
sudo apt-get install build-essential subversion git-core libncurses5-dev \
				    zlib1g-dev gawk flex quilt libssl-dev xsltproc \
				    libxml-parser-perl mercurial bzr ecj cvs unzip
```

2.下载OpenWrt源代码

```bash
# Lean's OpenWrt
git clone https://github.com/coolsnowwolf/lede
```

3.下载并安装所有可用的"feeds"

```bash
cd lede
./scripts/feeds update -a
./scripts/feeds install -a
```

4.运行下面的命令让OpenWrt编译系统检查你的编译环境中缺失的软件包:

```bash
make defconfig
make prereq
```

## 三.开始编译

1.配置编译选项

```bash
make menuconfig
```

出现下图面的界面，选择需要的安装配置信息
![build](http://7xseex.com1.z0.glb.clouddn.com/build.png "build")

> Y：选择Y，该软件将被编译，并且加入到你的目标固件里；
> M：选择M，该软件包将会被编译，但不会被放入固件里。在需要它的时候，可以用OPKG软件包管理器进行安装；
> N：选择N，该软件包将不会被编译，也不会被安装进固件。
> 方向键是移动光标
> 回车键是确认
> 空格键是选择，可以代替Y/M/N键的使用
> /:搜索

**备注**：记得勾选`Build the OpenWrt SDK`

> 经过多次失败的尝试，发现从官网下的`OpenWrt SDK`不能使用，总是编译出错，自己编译的确能够正常的使用，可能是使用的开发环境以及编译的版本不同吧

2.配置代理

```bash
export http_proxy='socks5://172.16.10.10:1080'
export https_proxy='socks5://172.16.10.10:1080'
```

3.开始编译

```bash
make download
make -j2 V=s
```

> 等待一段时间，便`/bin/$target`找到刷机的固件和SDK.

4.后期处理,清空多余的中间文件

```bash
make dirclean
make clean
make distclean
```

> 接下来将介绍使用`OpenWrt SDK`编译自己的软件包。

更多进阶的配置参考：
<http://www.jianshu.com/p/66c7b0969a31>
