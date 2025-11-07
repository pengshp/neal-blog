---
title: 使用 Checkinstall 编译打包Debian包
date: 2016-11-22T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---
有时我们需要把一个安装包编译成`xx.deb`格式的安装包，以实现编译一次就可以在其他系统上运行的目的。这时就可以使用`Checkinstall`,它是一个很好用的`linux`源码安装工具，是本文介绍使用Checkinstall 编译Debian包，以编译mosquitto为例.
<!-- more -->
#### 1.下载mosquitto源码包
    wget http://mosquitto.org/files/source/mosquitto-1.4.10.tar.gz

#### 2.解压源码包
    tar -zxvf mosquitto-1.4.10.tar.gz

#### 3.编译源码包
    cd mosquitto-1.4.10
    make

#### 4.使用`checkinstall`编译打包
    sudo apt-get install checkinstall
    sudo checkinstall --install=no --pkgversion 1.4.10
    # 后面两个参数表示打包后不安装，设置软件包版本。

#### 5.设置软件包参数
回车后会让你为这个软件包写一段描述，以空行或者`EOF`结束，接着会设置具体参数，比如下面的所示：

    *****************************************
    **** Debian package creation selected ***
    *****************************************
    
    软件包将用下面的值来创建：
    
    0 -  Maintainer: [ root@raspberrypi ]
    1 -  Summary: [ An Mosquitto Server(MQTT) by pengshp(mail:pengshp3@outlook.com) ]
    2 -  Name:    [ mosquitto ]
    3 -  Version: [ 1.4.10 ]
    4 -  Release: [ 1 ]
    5 -  License: [ GPL ]
    6 -  Group:   [ checkinstall ]
    7 -  Architecture: [ armhf ]
    8 -  Source location: [ mosquitto-1.4.10 ]
    9 -  Alternate source location: [  ]
    10 - Requires: [  ]
    11 - Provides: [ mosquitto ]
    12 - Conflicts: [  ]
    13 - Replaces: [  ]
    
    输入一个数字来改变它们，或按回车键继续：

一般默认即可，这样便可以在当前目录下生成`mosquitto_1.4.10-1_armhf.deb`。便可以保存下来再其他系统上也可安装。

#### 6.安装软件包
    sudo dpkg -i mosquitto_1.4.10-1_armhf.deb



