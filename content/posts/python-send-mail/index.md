---
title: Python 发送电子邮件
date: 2017-12-26T02:47:34+08:00
categories: [Python]
tags: [Python]
---
Email的历史比Web还要久远，直到现在，Email也是互联网上应用非常广泛的服务。本文介绍邮件的传递过程及使用Python实现邮件的发送。
<!--more-->
## 一、基本概念
假设我们自己的电子邮件地址是me@163.com，对方的电子邮件地址是friend@sina.com，现在我们用Outlook或者Foxmail之类的软件写好邮件，填上对方的Email地址，点“发送”，电子邮件就发出去了。这些电子邮件软件被称为MUA：Mail User Agent——邮件用户代理。
Email从MUA发出去，不是直接到达对方电脑，而是发到MTA：Mail Transfer Agent——邮件传输代理，就是那些Email服务提供商，比如网易、新浪等等。由于我们自己的电子邮件是163.com，所以，Email首先被投递到网易提供的MTA，再由网易的MTA发到对方服务商，也就是新浪的MTA。
Email到达新浪的MTA后，由于对方使用的是@sina.com的邮箱，因此，新浪的MTA会把Email投递到邮件的最终目的地MDA：Mail Delivery Agent——邮件投递代理。Email到达MDA后，就静静地躺在新浪的某个服务器上，存放在某个文件或特殊的数据库里，我们将这个长期保存邮件的地方称之为电子邮箱。
Email不会直接到达对方的电脑，因为对方电脑不一定开机，开机也不一定联网。对方要取到邮件，必须通过MUA从MDA上把邮件取到自己的电脑上。
所以，一封电子邮件的旅程就是： 
发件人 -> MUA -> MTA -> MTA -> 若干个MTA -> MDA <- MUA <- 收件人
发邮件时，MUA和MTA使用的协议就是SMTP：Simple Mail Transfer Protocol；
收邮件时，MUA和MDA使用的协议有两种：
* POP：Post Office Protocol，目前版本是3，俗称POP3；
* IMAP：Internet Message Access Protocol，目前版本是4，优点是不但能取邮件，还可以直接操作MDA上存储的邮件，比如从收件箱移到垃圾箱。

邮件客户端软件在发邮件时，会让你先配置SMTP服务器，也就是你要发到哪个MTA上。假设你正在使用163的邮箱，你就不能直接发到新浪的MTA上，因为它只服务新浪的用户，所以，你得填163提供的SMTP服务器地址：`smtp.163.com`，为了证明你是163的用户，SMTP服务器还要求你填写邮箱地址和邮箱口令，这样，MUA才能正常地把Email通过SMTP协议发送到MTA。 

## 二、SMTP发送邮件
SMTP是发送邮件的协议，Python内置对SMTP的支持，可以发送纯文本邮件、HTML邮件以及带附件的邮件。 
Python 对SMTP支持有`smtplib`和`email`两个模块，email负责构造邮件，smtplib负责发送邮件。
首先构造一个最简单的纯文本邮件：

```python
#!/usr/bin/env python3
from email.mime.text import MIMEText
import smtplib

msg=MIMEText('Hello,Send by Python..','plain','utf-8')
from_addr = input('From: ')
password = input('Password: ')

# 收件人地址
to_addr = input('To: ')

smtp_server = input('SMTP server: ')
server = smtplib.SMTP(smtp_server,25)
server.set_debuglevel(1)
server.login(from_addr,password)
server.sendmail(from_addr, [to_addr], msg.as_string())
server.quit()
```

构造MIMEText对象时，第一个参数就是邮件正文，第二个参数是MIME的subtype，传入'plain'表示纯文本，最终的MIME就是'text/plain'，最后一定要用utf-8编码保证多语言兼容性。
用`set_debuglevel(1)`就可以打印出和SMTP服务器交互的所有信息。SMTP协议就是简单的文本命令和响应。`login()`方法用来登录SMTP服务器，`sendmail()`方法就是发邮件，由于可以一次发给多个人，所以传入一个list，邮件正文是一个str，`as_string()`把MIMEText对象变成str。

测试后发现用上面的方法发送会有问题：
1、邮件没有主题
2、收件人名字显示不友好
3、收到邮件却显示不在不在收件人中

这是因为邮件主题、如何显示发件人、收件人等信息并不是通过SMTP协议发给MTA，而是包含在发给MTA的文本中的，所以，我们必须把From、To和Subject添加到MIMEText中，才是一封完整的邮件：

```python
#!/usr/bin/env python3
# Send email by Python3
from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.utils import parseaddr,formataddr
import smtplib

def _format_addr(s):
	name,addr = parseaddr(s)
	return formataddr((Header(name,'utf-8').encode(),addr))

from_addr = input('From: ')
password = input('Password: ')
to_addr = input('To: ')
smtp_server = input('SMTP server: ')

msg=MIMEText('Hello,send by Python...','plain','utf-8')
msg['From'] = _format_addr('Python爱好 <%s>' %from_addr)
msg['To'] = _format_addr('管理员 <%s>' %to_addr)
msg['Subject'] = Header('来自SMTP','utf-8').encode()

server=smtplib.SMTP(smtp_server,25)
server.set_debuglevel(1)
server.login(from_addr,password)
server.sendmail(from_addr, [to_addr], msg.as_string())
server.quit()
```

这样便可以发送一个正常额邮件。注意发件人邮箱的密码为客户端授权密码，可网页登录邮箱进入管理后台查看。

### zmail 收发邮件

zmail 是一个更好的用于发送邮件的库，使用简单

特征:

- 自动查找服务器地址及其端口。
- 自动使用合适的协议登录。
- 自动将python字典转换为MIME对象（带附件）。
- 自动添加邮件标题和本地名称，以避免服务器拒绝您的邮件。
- 轻松自定义邮件标题。
- 支持HTML作为邮件内容。
- 只需要python> = 3.5

1. 安装

```shell
pip3 install zmail
```

2. 发送邮件

```python
import zmail
server = zmail.server('yourmail@example.com’, 'yourpassword')
mail = {
    'from': 'Zmail <xxxx@163.com>',
    'subject': 'Success!',  # Anything you want.
    'content_text': 'This message from zmail!',  # Anything you want.
    'attachments': ['/Users/zyh/Documents/example.zip','/root/1.jpg'], # 附件
}
                      
if server.smtp_able():
	# Send mail
	server.send_mail('yourfriend@example.com',mail)
	# Or to a list of friends.
	server.send_mail(['friend1@example.com','friend2@example.com'],	mail)
```

3. 接收邮件

```python
import zmail
server = zmail.server('yourmail@example.com’, 'yourpassword')

if server.pop_able():
	# Retrieve mail
	latest_mail = server.get_latest()
	zmail.show(latest_mail)
```

4. 自定义邮件服务器

```python
server = zmail.server(
    'username',
    'password',
    smtp_host='smtp.163.com',
    smtp_port=994,
    smtp_ssl=True,
    pop_host='pop.163.com',
    pop_port=995,
    pop_tls=True
)
```

项目地址：<https://github.com/ZYunH/zmail>
