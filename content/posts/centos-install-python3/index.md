---
title: CentOS 安装 Python3
date: 2019-09-02T02:47:34+08:00
categories: [CentOS]
tags: [CentOS, Python]
---

CentOS 默认只安装 Python2 ,但未来主流的仍是 Python3，因此有必要在 CentOS 上安装 Python3，以下方式介绍在 CentOS 上编译安装Python3并解决安装后yum无法正常使用的问题。

<!--more-->

#### 1、下载Python源代码

```sh
wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tgz
```

#### 2、安装依赖包

```sh
yum -y install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
```

#### 3、解压源码包

```sh
tar -xzvf Python-3.6.1.tgz
cd  Python-3.6.1/
```

#### 4、把Python3.6安装到 /usr/local 目录

```sh
./configure  --prefix=/usr/local/
make
make altinstall
```

此处使用 `make altinstall` 安装是为了使Python2 和Python3 能共存，具体可见压缩包里的`README`文件.

python3.6程序的执行文件：`/usr/local/bin/python3.6`
python3.6应用程序目录：`/usr/local/lib/python3.6`
pip3的执行文件：`/usr/local/bin/pip3.6`
pyenv3的执行文件：`/usr/local/bin/pyenv-3.6`

#### 5、更改Python链接

当在命令行输入Python时默认打开的是Python2版本，现在把它改为Python3；

```sh
cd/usr/bin
mv  python python.backup
ln -s /usr/local/bin/python3.6 /usr/bin/python
ln -s /usr/local/bin/python3.6 /usr/bin/python3

rm -rf /usr/bin/python2
ln -s /usr/bin/python2.6 /usr/bin/python2
```

#### 6、更改yum脚本的Python依赖

现在如果执行yum相关的命令，会发现不能正常执行，因为yum是用Python2开发的，然而此时Python默认指向的是Python3,二者的语法不兼容，所以会出现报错，为了解决此问题，需要修改yum相关命令的python依赖环境

```sh
$ cd /usr/bin
$ ls yum*
yum yum-config-manager yum-debug-restore yum-groups-manager yum-builddep yum-debug-dump yumdownloader
```

使用`vim`更改以上文件头为
`!/usr/bin/python` 改为 `#!/usr/bin/python2`

这样便能很好的解决Python的版本问题及yum不能正常使用的问题。

### 附：快捷办法

```sh
$ yum install epel-release
$ yum makecache
$ yum install -y python36
```
