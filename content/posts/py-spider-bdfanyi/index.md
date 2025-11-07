---
title: Python3爬虫百度翻译查词
date: 2018-10-20T02:47:34+08:00
categories: [Python]
tags: [Python]
---

百度翻译是一个不错的查词翻译网站，本文介绍使用Python3爬虫实现百度翻译查词的效果，主要的依赖是`requests`库。

<!--more-->

### 测试环境
- Windows10 x64
- PyCharm 2018.2
- Python3.6.4

### 依赖
- requests
请求的API：<https://fanyi.baidu.com/sug>

### 源代码

```py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

# &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
#   @File Name:    bdfy.py
#   @Description:  百度翻译
#   @Author:       pengshp<pengshp3@outlook.com>
#   @date:         2018/10/14 0014
# &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&


import requests


def baiduFanyi(word):
    """百度翻译单词查询"""
    url = 'https://fanyi.baidu.com/sug'
    header = {
        "User-Agent": "Mozilla/5.0"
    }
    data = {
        'kw': word
    }
    response = requests.post(url, headers=header, data=data)
    resData = response.json()  # 把返回的json数据转为字典
    result = resData['data'][0].get('v')
    return result


if __name__ == '__main__':
    word = input('请输入你要查询的单词：')
    print(baiduFanyi(word))

```

主要是使用requests库的`post`请求方法请求百度翻译的`API`，把返回的`json`数据转为字典，再提取数据。

### 运行测试

```sh
~$ python3 bdfy.py
请输入你要查询的单词：python
n. 巨蛇，大蟒;
```

### 使用`Google Fire`封装为`CLi`

```python
#!/usr/bin/env python3
# -*- coding:UTF-8 -*-

# @Version: v1.0
# @Author: pengshp<pengshp3@outlook.com>
# @License: Apache Licence 2.0
# @Description: 百度翻译API
# @File: bdfy.py
# @Date: 10月 26,2018

import requests
import fire


def baiduFanyi(word):
    """百度翻译单词查询"""
    url = 'https://fanyi.baidu.com/sug'
    header = {
        "User-Agent": "Mozilla/5.0"
    }
    data = {
        'kw': word
    }
    response = requests.post(url, headers=header, data=data)
    resData = response.json()  # 把返回的json数据转为字典
    result = resData['data'][0].get('v')
    return result


if __name__ == '__main__':
    fire.Fire(baiduFanyi)

```

### 使用示例

```powershell
PS F:\code\myfire> pipenv run python bdfy.py -- --help
Type:        function
String form: <function baiduFanyi at 0x000001A023011E18>
File:        bdfy.py
Line:        15
Docstring:   百度翻译单词查询

Usage:       bdfy.py WORD
             bdfy.py --word WORD
PS F:\code\myfire> pipenv run python bdfy.py python
n. 巨蛇，大蟒;
PS F:\code\myfire> pipenv run python bdfy.py 你好
[nǐ hǎo] hello; hi; How do you do!;
```



### 参考

1、[百度翻译](https://fanyi.baidu.com)
2、[requests中文指南](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html)
