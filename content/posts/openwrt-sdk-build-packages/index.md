---
title: OpenWrt-SDK编译自己的ipk包
date: 2016-08-05T02:47:34+08:00
categories: [OpenWrt]
tags: [OpenWrt,SDK]
---
本问介绍如何使用OpenWrt编译自定义的ipk包。
<!--more-->

上一篇文章介绍了编译openwrt固件和SDK，现在介绍一下使用**OpenWrt-SDK**编译自己的安装包，此次以**OpenWrt-SDK-brcm2708-bcm2709_gcc-4.8-linaro_uClibc-0.9.33.2_eabi.Linux-i686**为工具链编译**sysuh3c**[中山大学东校区校园网认证]

## 一.开发环境 
#### 1.Ubuntu14.10 32bit
#### 2.OpenWrt-SDK

## 二.编译设置 
1.新建一个目录下载软件包 

	$ mkdir sysuh3c
	$ git clone 

2.把下载的软件包复制到**OpenWrt-SDK**的**package**目录下

	$ cp sysuh3c OpenWrt-SDK-brcm2708-bcm2709_gcc-4.8-linaro_uClibc-0.9.33.2_eabi.Linux-i686/package

3.修改**sysuh3c**的**Makefile**配置文件

	$ cd sysuh3c
	$ vim Makefile

主要修改系统架构和版本等信息，如下图所示，这里改为树莓派2B的架构bcm2709,这个工具链也可以用来编译树莓派1代【brcm2708】的ipk包

![rspi2-makefile](http://7xseex.com1.z0.glb.clouddn.com/rspi2-makefile.png)

在package目录下执行tree查看目录树应该是这样

```sh
	.
	├── Makefile
	└── sysuh3c
	    ├── Makefile
	    ├── README.md
	    └── src
	        ├── eapauth.c
	        ├── eapauth.h
	        ├── eapdef.h
	        ├── eaputils.c
	        ├── eaputils.h
	        ├── main.c
	        ├── Makefile
	        └── sysuh3c.conf

	2 directories, 11 files
```
注意修改的是**sysuh3c**目录下的**Makefile**，其他文件不要作任何修改

## 三.编译 
1.回到**OpenWrt-SDK-brcm2708-bcm2709_gcc-4.8-linaro_uClibc-0.9.33.2_eabi.Linux-i686**目录下，执行下面的命令

	$ make 

2.编译好的ipk安装包在**bin/brcm2708/packages/base**

	$ cd bin/brcm2708/packages/base 
	$ ls
	sysuh3c_2.0-0_bcm2709.ipk

## 三.安装ipk包 
使用winscp将**sysuh3c_2.0-0_bcm2709.ipk**上传到路由器的/tmp目录下，使用[Putty](http://putty.org) ssh登陆到路由器执行下面的命令

	$ cd /tmp 
	$ opkg install sysuh3c_2.0-0_bcm2709.ipk

> OpenWrt-SDK一般自己编译一个，官网下载的编译时有问题，编译的环境最好选择32bit的Ubuntu系统。不然很容易出问题，编译OpenWrt-SDK和固件时最好别选过多的其他功能或者包，不然很容易出问题导致编译失败，需要的安装包可以编译好后下载安装或者自己编译。

