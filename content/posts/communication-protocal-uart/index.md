---
title: 通信协议 UART 总结
date: 2017-05-29T02:47:34+08:00
categories: [IoT]
tags: [IoT]
---

UART是通用异步收发器（异步串行通信口）的英文缩写，主要包括RS232,RS485等接口标准和总线标准规范。还有一种是 USART(同步异步串行通信口)，但使用较少。

<!--more-->

### UART 简介

`UART` 协议主要包含三个引脚 串行发送引脚(TXD),串行接受引脚(RXD),电源参考地(GND)。当两个单片机进行通信时，电源地GND接在一起，然后单片机1的 `TXD` 引脚接单片机2的 `RXD` 引脚，即此路为单片机1发送二单片机2接收的通道；单片机1的 `RXD` 接到单片机2的 `TXD` 引脚，即此路为单片机2发送而单片机1接收的通道。如下所示：
![uart.png](http://7xseex.com1.z0.glb.clouddn.com/2017052917271uart.png)
当单片机1给单片机2发送数据时，比如发送一个 0xE4,二进制表示是11100100，是低位先发送，高位后发送。

### 波特率

在 `UART` 通信中有一个波特率的概念，即发送二进制数据位的速率，习惯用 `baud` 表示。当两个单片机进行通信是必须保证两个单片机的波特率一致。常用的波特率有9600，115200。在串口助手中设置波特率时一般设置为`9600`，校验位选N，数据位8，停止位1。
一个完整的 `UART` 数据帧由一个起始位(0)，8个数据位，一个停止位(1)构成.
![uart-frame.jpg](http://7xseex.com1.z0.glb.clouddn.com/2017052991572uart-frame.jpg)

### 串口调试工具 Minicom

`Minocom` 是 `Linux/OS X` 下的一个串口调试命令行工具，在 `Linux/OS X` 下进行串口调试时一般都会使用此工具。

#### 1、安装

```sh
# Ubuntu
sudo apt install minicom
# MAC
brew install minicom
```

#### 2、设置

安装完成后第一次使用需要设置，执行`sudo minicom -s`进行设置，设置为如下所示：
![uart-setup.png](http://7xseex.com1.z0.glb.clouddn.com/2017052982614uart-setup.png)
按对应项的字母进入相应的编辑选项，按照相应的说明编辑完成后回车，保存为默认配置，这样下次使用时直接使用`minicom`命令就可以使用。
