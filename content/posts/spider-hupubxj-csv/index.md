---
title: 爬取虎扑网步行街帖子信息并写入CSV文件
date: 2018-10-19T02:47:34+08:00
categories: [Python]
tags: [Python]
---

最近在学Python网络爬虫，自己写了一些小项目，本次介绍使用requests和BeautifulSoup4库爬取虎扑网的帖子信息并写入到CSV文件。

<!--more-->

[虎扑步行街](https://bbs.hupu.com/bxj)的帖子很有意思，记得大学舍友经常看，遇到有意思的会和我们讲，本次爬取步行街帖子的*标题，URL，作者，发帖时间，回帖数，浏览数，最后回帖时间，最后回帖人*信息并写入到CSV文件。

### 环境
* Python 3.6.4
* PyCharm 2018
* Windows10 x64

### 依赖：
- requests == 2.19.1
- beautifulsoup4 == 4.6.3
- lxml == 4.2.4

### 源代码

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

# &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
#   @Version: v1.0
#   @License: Apache Licence 2.0
#   @File Name: hupubxj.py
#   @Description: 爬取虎扑步行街的帖子信息并写入csv文件
#   @Author: pengshp <pengshp3@outlook.com>
#   @Date: 2018/10/19 0019
# &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

import requests
from bs4 import BeautifulSoup
import bs4
import csv


def getHTML(link):
    """获取网站HTML"""
    kv = {'User-Agent': 'Mozilla/5.0'}
    try:
        r = requests.get(link, headers=kv)
        r.raise_for_status()
        html = r.text
    except requests.HTTPError:
        print("爬取异常")
    return html


def getData(html):
    """获取帖子的详细信息"""
    data_list = []
    soup = BeautifulSoup(html, 'lxml')
    ul = soup.find_all('ul', class_="for-list")
    for li in ul[0].children:
        if isinstance(li, bs4.element.Tag):
            title_box = li.select('a[class="truetit"]')
            title = title_box[0].string
            url = 'https://bbs.hupu.com%s' % title_box[0]['href']
            author = li.select('a[class="aulink"]')[0].string
            time = li.select('a[style="color:#808080;cursor: initial; "]')[0].string
            relay_view = li.select('span[class="ansour box"]')[0].string
            relay = relay_view.split('/')[0].strip()
            view = relay_view.split('/')[1].strip()
            endreply = li.select('div[class="endreply box"]')[0].a.string
            endauthor = li.select('div[class="endreply box"]')[0].span.string
            data_list.append([title, url, author, time,
                              relay, view, endreply, endauthor])
    return data_list


if __name__ == '__main__':
    link = 'https://bbs.hupu.com/bxj'
    page = getHTML(link)
    datas = getData(page)
    for data in datas:
        with open('bxj.csv', 'a+', encoding='utf-8', newline='') as f:
            w = csv.writer(f)
            w.writerow(data)
    print("数据写入完成！")
```

### 思路：

先使用`requests`库获取网页的`HTML`，再使用`bs4`库解析网页中的数据，这里是难点，需要对照源码找到对应的标签，并提取所需的信息,把单个帖子所有的信息存为一个`List`，再把每个List加入到总的数据List中，相当于一个二维数组。写数据时一个帖子的所有信息作为一行写入`bxj.csv`.

### 结果

打开`bxj.csv`查看数据。
![bxjdata](/images/bxj.png "top")
