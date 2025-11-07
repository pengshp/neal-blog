---
title: Python 的 IP 地址处理模块 IPy
date: 2017-12-29T02:47:34+08:00
categories: [Python]
tags: [Python]
---
IP 地址规划是网络设计中很重要的一个环节，在此过程中，免不了要计算大量的IP 地址，包括网段、子网掩码、广播地址、子网数、IP类型等。Python 提供了一个强大的第三方模块`IPy`,可以辅助我们高效的完成 IP 的规划工作。
<!--more-->
#### 1、安装 IPy 模块
    pip3 install ipy

#### 2、IP 地址、网段的基本数理
IPy 模块包含一个`IP`类,可以方便的处理绝大部分格式为 `IPv4` 和 `IPv6` 格式的网络和地址。

```python
>>> from IPy import IP
>>> IP('10.0.0.0/8').version()
4
>>> IP('::1').version()
6
```

通过指定的网段输出该网段的IP个数及所有的IP地址清单

```python
#!/usr/bin/env python3
from IPy import IP
ip=IP('192.168.22.0/24')
print(ip.len())
for x in ip:
    print(x)
```

反向地址解析,IP类型，IP转换

```python
>>> from IPy import IP
>>> ip = IP('192.168.22.167')
>>> ip.reverseNames()      #反向解析
['167.22.168.192.in-addr.arpa.']
>>> ip.iptype()
'PRIVATE'
>>> IP('8.8.8.8').iptype() #8.8.8.8位公网
'PUBLIC'
>>> IP('8.8.8.8').int()    #转为整型格式
134744072
>>> IP('8.8.8.8').strHex() #转为十六进制
'0x8080808'
>>> IP('8.8.8.8').strBin() #转为二进制
'00001000000010000000100000001000'
>>> print(IP(0x8080808))
8.8.8.8
```

IP 方法也支持网络地址的转换，如IP和掩码

```python
>>> from IPy import IP
>>> ip=IP('192.168.22.0/255.255.255.0',make_net=True)
>>> print(ip)
192.168.22.0/24
```

通过strNormal方法指定不同参数值以定制不同输出类型的网段，输出类型为字符串。

```python
>>> IP('192.168.22.0/24').strNormal(0)
'192.168.22.0'
>>> IP('192.168.22.0/24').strNormal(1)
'192.168.22.0/24'
>>> IP('192.168.22.0/24').strNormal(2)
'192.168.22.0/255.255.255.0'
>>> IP('192.168.22.0/24').strNormal(3)
'192.168.22.0-192.168.22.255'
```

#### 3、多网络计算方法
判断IP地址和网段是否包含于同一网段中

```python
>>> '192.168.22.100' in IP('192.168.22.0/24')
True
>>> IP('192.168.22.0/24') in IP('192.168.0.0/16')
True
```

判断两个网段是否存在重叠，采用IPy 提供的 `overlaps` 方法

```python
>>> IP('192.168.0.0/23').overlaps('192.168.1.0/24')
1   #返回1代表存在重叠
>>> IP('192.168.0.0/23').overlaps('192.168.22.0/24')
0   #返回0代表不存在重叠
```

#### 4、综合应用
一个自动识别IP地址、子网、方向解析、IP类型的脚本

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from IPy import IP
ip_s = input(‘请输入IP地址或者网段地址: ‘)
ips = IP(ip_s)   #定义元素
if len(ips) > 1:  #如果len出来的数字大于1，那么就是一个网段
	print(‘网络地址: %s’ % ips.net())
	print(‘子网掩码: %s’ % ips.netmask())
	print(‘网络广播地址: %s’ % ips.reverseNames() [0])
	print(‘网络子网数: %s’ % len(ips))
else:   ###否则就是一个地址
	print(‘IP反向解析: %s’ % ips.reverseNames() [0])
	print(‘十六进制地址: %s’ % ips.strHex())
	print(‘二进制地址: %s’ % ips.strBin())
	print(‘地址类型: %s’ % ips.iptype())
```



