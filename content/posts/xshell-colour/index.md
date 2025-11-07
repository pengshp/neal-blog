---
title: Ｘshell自定义配色方案
date: 2016-12-02T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---

经常使用xshell来操作Linux的朋友都知道，xshell默认的配色方案很不好看，看着都不舒服，所以从网上找了几个比较好看的配色方案，与各位分享！
<!--more-->

##### 一.方案一:Solarized Dark
```sh
    [Names]
	count=1
	name0=Solarized Dark
	[Solarized Dark]
	text(bold)=839496
	magenta(bold)=6c71c4
	text=00ff40
	white(bold)=fdf6e3
	green=859900
	red(bold)=cb4b16
	green(bold)=586e75
	black(bold)=073642
	red=dc322f
	blue=268bd2
	black=002b36
	blue(bold)=839496
	yellow(bold)=657b83
	cyan(bold)=93a1a1
	yellow=b58900
	magenta=dd3682
	background=042028
	white=eee8d5
	cyan=2aa198
```
#### 二.方案二:自定义方案
```sh
    [myxshell]
	text(bold)=e9e9e9
	magenta(bold)=ff00ff
	text=00ff80
	white(bold)=fdf6e3
	green=80ff00
	red(bold)=ff0000
	green(bold)=3c5a38
	black(bold)=808080
	red=ff4500
	blue=00bfff
	black=000000
	blue(bold)=1e90ff
	yellow(bold)=ffff00
	cyan(bold)=00ffff
	yellow=c0c000
	magenta=c000c0
	background=042028
	white=c0c0c0
	cyan=00c0c0
	[Names]
	count=1
	name0=myxshell
```
#### 三.使用方法
 1.以上方案可以存入txt文本中，然后修改后缀名为xcs再导入就可以了。

 2.具体颜色就不上图了，各位自己导入看吧，在xshell界面右键选择配色方案-->导入，就可以看到了。

