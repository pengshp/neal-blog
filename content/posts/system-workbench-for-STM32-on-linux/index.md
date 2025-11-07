---
title: Linux下搭建STM32开发环境
date: 2016-02-12T02:47:34+08:00
categories: [STM32]
tags: [STM32,Linux]
---

摘要：这个系统工具链叫做SW4STM32,是一个免费的基于Eclipse的multi-OS软件开发环境，它支持所有的STM32开发板及有关的开发板。这个产品是第三方提供而不是ST官方，最新的消息可以访问第三方的网站：<http://www.ac6.fr>
<!-- more -->
### 主要特点：
* 支持全系列的STM32开发板，包括STM32 Nucleo,Discovery套件和Evaluation.以及STM32固件（标准库或者STM32Cube HAL）
* GCC C/C++编译器
* GDB-based 调试器
* 带团队管理工具的Eclipse IDE
* 兼容Eclipse插件
* ST-LINK 支持
* 没有代码量的限制
* Multiple OS支持：Windows, Linux, OS X

### 一.在Ubuntu GNOME 16.04上安装开发环境
#### 1.安装之前需要安装java运行环境，进入命令行

```sh
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```

#### 2.下载安装脚本
```bash
# 32bit
$ wget http://www.ac6-tools.com/downloads/SW4STM32/install_sw4stm32_linux_32bits-latest.run  
# 64bit
$ wget http://www.ac6-tools.com/downloads/SW4STM32/install_sw4stm32_linux_64bits-latest.run  
```
#### 3.对于Ubuntu 64-bit,需下载而外的工具

    $ sudo apt-get install libc6:i386 lib32ncurses5

#### 4.为了启动GUI模式，需要安装`gksu`

    $ sudo apt-get install gksu

#### 5.进入下载目录执行

    $ chmod +x install_sw4stm32_linux_64bits-v1.8.run
    $ ./install_sw4stm32_linux_64bits-v1.8.run

#### 6.安装完成后点击`Help`-->`Update`更新最新的固件库。

### 二.建立工程项目
依次点击`File`-->`Project`-->`C Project`-->`C Project`-->`Next`,j加上工程的名字，Project type选择`Ac6 STM32 MCU Project`,Toolchains默认，再`Next-->Next`,Series里选择STM32的主型号，Board里选择具体的型号。再``Next`-->`Finish`.

> 到此整个搭建过程就完成了，接下来就是使用它来实现你的想法，Enjoy it!!

#### 参考：
1.<http://www.st.com>
2.<http://www.openstm32.org>



