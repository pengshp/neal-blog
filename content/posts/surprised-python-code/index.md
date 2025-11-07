---
title: 令人震惊的单行Python代码
date: 2016-07-12T02:47:34+08:00
categories: [Python]
tags: [Python]
---
本文主要介绍几条简短的但功能强大的`Python`代码，主要是`Python3`环境，建议在`IPython3`环境中运行。
<!--more-->
#### 1.让列表中的每个元素都乘以２
	print (map(lambda x: x*2,range(1,11)))
#### 2.求列表中的所有元素的和
	print　(sum(range(1,1001)))
#### 3.判断一个字符串中是否存在某些词
	wordlist = ["scala","akka","play frame work","sbt","typesafe"]
	tweet = "This is an example tweet talking about scala and sbt."
	print (map(lambda x: x in tweet.split(),wordlist))
#### 4.读取文件
	print (open("ten_one_liners.py").readlines())
#### 5.祝你生日快乐
	print (map(lambda x: "Happy Birthday to "+ ("you" if x != 2 else "dear Name"),range(4)))
#### 6.过滤列表中的数值
	print (reduce(lambda(a,b),c): (a+[c],b) if c>60 else (a,b + [c]),[49,58,76,82,88,90],([],[]))
#### 7.找出列表中最大或者最小的一个数字
	print (min([14,35,-7,46,98]))
	print (max([14,35,-7,46,98]))
#### 8.并行处理
	import multiprocessing 
	import math
	print(list(multiprocessing.Pool(processes=4).map(math.exp,range(1,11))))

