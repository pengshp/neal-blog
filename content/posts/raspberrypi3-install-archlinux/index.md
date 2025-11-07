---
title: 树莓派2/３安装Archlinux教程
date: 2016-03-28T02:47:34+08:00
categories: [RaspberryPi]
tags: [RaspberryPi, Linux]
---

本文主要介绍最新的树莓派2/３安装Ａrchlinux系统的主要步骤。树莓派官方最近发布了最新的树莓派３，与上一代树莓派２B+相比，标配了蓝牙和wifi，主频达到1.2GHz,速度更快也更加流畅了。拿到手后赶快体验了一把。

<!--more-->

![arch](/images/archlinux.png "Arch Linux")
### 一.准备工作
* 一台Linux操作系统的电脑
* 一张８Ｇ的ＳＤ卡
* 树莓派３主板

### 二.下载安装
#### 1.插入SD卡，首先卸载卡，执行下面的命令

    $ sudo umount /media/EAN-7EDS

后面的EAN-7EDS根据自己的情况，插入一张卡的话，一般按Tab键补齐
#### 2.在Ubuntu中执行下面命令查看挂载的位置我的是/dev/sdb

    $ sudo fdisk -l

#### 3.终端执行下面命令下载系统文件

    $ sudo mkdir archlinux
    $ cd archlinux
    $ wget http://mirrors.ustc.edu.cn/archlinuxarm/os/ArchLinuxARM-rpi-2-latest.tar.gz

#### 4.为SD卡分区

    $ sudo fdisk /dev/sdb

>   输入 o 并回车，这将会删除所有分区
    输入 p 并回车，这将会列出所有分区，此时应该没有任何分区
    输入 n 并回车，创建新分区，引导分区
    输入 p 并回车，新分区为主分区
    输入 1 并回车，分区序号是1
    按键盘回车，默认初始扇区
    输入 +100M 并回车，设置终止扇区
    输入 t 并回车，再输入 c 并回车，设置该分区文件系统格式为Fat32
    输入 n 并回车，创建新分区，根分区
    输入 p 并回车，新分区为主分区
    输入 2 并回车，分区序号是2
    按键盘回车，默认初始扇区
    按键盘回车，默认终止扇区
    输入 w 并回车，写入设置

#### 4.格式化分区

    $ sudo mkfs.vfat /dev/sdb1
    $ sudo mkfs.ext4 /dev/sdb2

#### 5.创建挂载位置并挂载分区

    $ sudo mkdir boot root
    $ sudo mount /dev/sdb1 boot
    $ sudo mount /devsdb2 root

#### 6.解压文件到root目录

    $ sudo apt-get install bsdtar
    $ sudo badtar -xpf ArchLinuxARM-rpi-2-latest.tar.gz -C root

#### 7.清空缓存

    $ sudo sync

#### 8.把boot中的文件移动到第一个分区

    $ mv root/boot/* boot

#### 9.卸载两个分区

    $ sudo umount boot/ root/

#### 10.取下卡，插入树梅派上电即可启动

>ssh登录用户名：alarm
 密码：alarm

> 备注：到目前位置，针对树莓派3B　Cotex-A53 64位架构的镜像包还没有发布，暂时使用的使树莓派２B的ArmV7架构,不够根据官网的描述，未来将会通过更改64bit的软件源实现更新！
到此就装完了，一些基本配置参考：
[Raspberry Pi 简体中文](https://wiki.archlinux.org/index.php/Raspberry_Pi)

