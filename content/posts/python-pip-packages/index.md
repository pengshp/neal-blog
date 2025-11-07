---
title: Python的包管理工具---pip
date: 2017-03-01T02:47:34+08:00
categories: [Python]
tags: [Python]
---
本文介绍Python的包管理工具Pip.
<!--more-->

### 一.下载安装pip
到官网`https://github.com/pypa/pip/releases`下载最新的源码包，解压，进入包目录，执行下面的命令

    sudo python3 setup.py install

### 二.更新pip版本
有时使用pip时会提示pip版本太旧，需要更新，可执行

    python3 -m pip3 install -U pip3

### 三.常用命令
```py
pip3 install [package name] # 安装软件包
pip3 uninstall [package name] # 卸载软件包
pip3 install --upgrade [package name] # 升级软件包
pip3 list --outdated # 列出可更新的软件包
pip3 download [package name] # 下载软件包
pip3 show [package name] # 显示已安装软件包的信息
pip3 search [package name] # 在pip源中搜寻相关的软件包
```

### 四.修改pip源
官方的pip源下载速度较慢，有时会出现超时不能下载的情况。这时就需要修改默认的pip源，建议使用阿里云的`pip`源。修改`pip.conf`文件

    vim ~/.pip/pip.conf
    [global]
    trusted-host=mirrors.aliyun.com
    index-url=https://mirrors.aliyun.com/pypi/simple/

保存退出，重启终端，文件不存在时需要新建。
### 五.一次性更新所有的软件包
新建一个`upgrade_packages.py`脚本，脚本内容如下所示：
```py
import pip
from subprocess import call

def upgrade_all():
    print("Upgrading all outdated packages......")
    for dist in pip.get_installed_distributions():
        call("pip install --upgrade " + dist.project_name, shell=True)

if __name__ == '__main__':
    upgrade_all()
    print("Upgrade finished!")
```
然后在终端中执行：

    # Mac OS
    sudo -H python3 upgrade_packages.py 

> 当长时间未更新时，Python的许多软件包就会有所更新，为了使用软件包最新的特性，便可以使用此方法一次性更新所有的软件包。
> 有些软件包的安装需要管理员权限，要使用`sudo`

### 六.参考链接
* https://pip.pypa.io/en/latest/user_guide.html
* http://www.pip-installer.org/en/latest/configuration.html

