---
title: Raspberrypi3的UbuntuMate16.04上安装最新qt5
date: 2016-05-01T02:47:34+08:00
categories: [RaspberryPi]
tags: [RaspberryPi]
---
本文主要介绍在树莓派３上的UbuntuMate16.04上安装最新的ＱＴ５开发环境！加上简单的配置和运行一个hello word程序!本教程也适用于Raspbian系统。
<!--more-->
### 一.安装qt5
运行下面几条命令下载安装最新的qt5

    sudo apt-get update
    sudo apt-get install qt5-defalut
    sudo apt-get install qtcreator

### 二.配置qt5
从菜单栏中打开qtcreator，依此选择工具-->选项-->构建和运行-->编辑器-->添加-->GCC,浏览里选择/usr/bin/gcc或者交叉编译/usr/bin/arm-linux-guneabihf-gcc,点击Apply-->OK.

### 三.简单测试
回到欢迎菜单，选择New Project-->Nor-QT Project-->Plain C++ Application,选择choose,填写项目名和路径，下一步，Build system选择默认的```qmake```,再一直下一步，完成。写个简单的```hello word```程序测试：
![qt5](/images/qt5.png "Qt5")
点击左下角小锤子的图标构建（快捷键```Ctrl+B```）编译，点击绿色的三角形图标运行（快捷键```Ctrl+R```）运行程序。如果没有错误就会打开终端显示运行结果。
到此配置结束，enjoy it!开启你的ＱＴ之旅吧！
