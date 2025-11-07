---
title: "Shell+Curl实现网页爬虫"
date: 2016-09-08T02:47:34+08:00
categories: [Shell]
tags: [Shell,Curl]
---

使用shell和curl监测海贼王的更新并发邮件到邮箱！
<!--more-->

### 一.项目描述
为了能进入导师实验室，师兄分给我一个项目，用shell脚本和curl实现一个网络爬虫，主要功能是检测某个网站海贼王的更新信息，当有最新剧集更新时发送邮件提醒。作为对我的考验，刚开始我没有思路，学了一段时间的shell脚本，又从网上查了一些资料，但感觉没思路；后来问了一个软件学院的师兄，可以用正则表达式匹配出最大值，与上一次的值比较，大于则发送邮件。

### 二.源码实现

```bash
#Function: 监测海贼王更新并提醒
#Author: pengshp
#Time: 2015-9
#! /bin/bash

URL="http://www.fzdm.com/manhua/002/"
LOG_DIR=${HOME}/logs

#发送邮件函数
smail(){
mail -s "更新提醒" $MAIL << EOF
==================================
Report Time: `date +"%F %T"`
Current User: `whoami`
海贼王已经有更新，请打开下列链接前往观看!
风之动漫：
http://www.fzdm.com/manhua/002/
===================================
EOF
}

read -p "Please input your 163 mail: " MAIL
Result=`curl -o /dev/null -s -m 10 --connect-timeout 10 -w %{http_code} $URL`
Test=`echo $Result`

#监测网站是否可以正常访问
if [ "$Test" == "200" ]
then
	if [ ! -f $LOG_DIR ]
	then
		mkdir -p $LOG_DIR
	fi
	cd $LOG_DIR
	touch log.txt
	let Value1=800  #现在最新是800话
	while true
	do
		curl -s -o page.html http://www.fzdm.com/manhua/002/
		#利用正则表达式获取HTML文件中最新的剧集的值
		let Value=$(cat page.html |grep 'title="海贼王[0-9].*"' |sed -n '1p' |awk -F '"' '{print $4}' |cut -d '/' -f1)
		if [ $Value -gt $Value1 ]
		then
			smail
			echo "successfully! `date` " >> log.txt
			let Value1=Value    #更新最新的剧集
		fi
		rm page.html
		sleep 300
	done
fi

```

> 程序中用到mail命令，需要自己配置相应的环境，把要发送的内容封装起来，以`EOF`结尾。接受的邮箱可以自己定义。

