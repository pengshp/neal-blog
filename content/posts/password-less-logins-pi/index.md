---
title: 免密码登录树莓派
date: 2016-08-20T02:47:34+08:00
categories: [Linux]
tags: [RaspberryPi]
---

摘要：平时在终端中登录树莓派每次都要输密码等，比较麻烦，因此找到一个比较简单的登录方法，只需要在终端输入`ssh pi`便可以登录，下面介绍配置方法.
<!-- more -->
### 一.测试环境：
* 系统：MacOS X
* shell: zsh
* 登录：RaspberryPi3B

### 二.设置方法
1.用`ssh-keygen`生成ssh密钥

```shell
$ ssh-keygen -t rsa -b 1024
$ cat ~/.ssh/id_rsa.pub
```

2.安装`ssh-copy-id`

```shell
$ brew install ssh-copy-id    # MacOS X
$ sudo apt install ssh-copy-id  # Ubuntu/Debian
$ yum install -y ssh-copy-id  # CentOS
```

3.把公钥上传到树莓派

```shell
$ ssh-copy-id pi@192.168.10.195  #改为树莓派的IP地址
```

再输入登录密码，回车便可以了。

4.在家目录下新建`.ssh/config`

```shell
$ vim ~/.ssh/config
Host pi
    Hostname 192.168.10.195
    user pi
    port 22
    IdentityFile ~/.ssh/id_rsa
```

加入以下所示的内容即可，pi的名字可自己随便起，保存退出。
![sshpi](/images/sshpi.png "ssh")

5.登录树莓派
现在在终端中执行`ssh pi`便可以登录。

## 附：解决xshell密钥验证登录需要输入密码的问题

```shell
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
```





