---
title: CentOS7基本配置
date: 2018-01-02T02:47:34+08:00
categories: [CentOS]
tags: [CentOS, Linux]
---

CentOS是目前主流的服务器发行版本，本文介绍CentOS初始化安装后的一些基本配置，版本为最新的CentOS7.4.安装时勾选安装基本的开发组件。

<!--more-->

### 1、配置网络

```sh
sudo nmtui  #  配置IP
sudo systemctl start network
sudo systemctl enable network
```

### 2、修改yum源

```sh
$ mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
$ yum clean all
$ yum makecache
```

### 3、更新软件源

```sh
$ yum -y update
$ yum -upgrade
```

### 4、安装pip

```sh
yum -y install epel-release
yum -y install python3-pip
sudo pip3 install --upgrade pip
```

### 5、安装 zsh + oh my zsh

```sh
$ yum install zsh git
$ git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
$ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
$ vim ~/.zshrc
export PATH=$HOME/bin:/usr/local/bin:$PATH
ZSH_THEME="sammy"
export ZSH=$HOME/.oh-my-zsh
plugins=(
	zsh-syntax-highlighting
	zsh-autosuggestions
	git
	)
export MANPATH="/usr/local/man:$MANPATH"
export ARCHFLAGS="-arch x86_64"

$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
$ git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions

$ chsh -s /bin/zsh

```

### 6、安装vim

```sh
$ yum -y install vim
```

#### 配置vim

```sh
[root@CentOS7] ~$ vim /etc/vim/vimrc
" 开启语法高亮
syntax on
" 检测文件类型
filetype on
" 设置在Vim中可以使用鼠标，防止终端无法拷贝
if has('mouse')
    set mouse-=a
endif
" 显示当前行号和列号
set ruler
" 在状态栏显示正在输入的命令
set showcmd
" 左下角显示当前Vim模式
set showmode
" 显示行号
"set number
" 设置tab宽度
set tabstop=4
" 智能自动缩进
set smartindent
" 设置自动对齐空格数
set shiftwidth=4
" 设置编码方式
set encoding=utf-8

set helplang=cn
set shiftwidth=4
set softtabstop=4
set magic
set cursorline

set hlsearch
set incsearch
" set autoindent

" 使用空格代替tab
set expandtab
set smarttab
```

### 7、安装常用软件

```sh
$ yum install -y net-tools git htop lrzsz
```

### 8、安装中文支持包

```sh
$ yum -y groupinstall chinese-support
```

### 9、安装基本编译环境

```sh
$ yum -y install gcc gcc-c++ make
```

### 10、关闭SELinux

```sh
$ vim /etc/selinux/config
SELINUX=disabled
```

### 11、设置主机名

```sh
$ hostnamectl set-hostname CentOS7
```

### 12、设置时区

```sh
$ timedatectl set-timezone Asia/Shanghai
```

### 13. 安装vm-tools

如果你在VMware相关的软件中安装`CentOS`则安装下面的软件。

```sh
~$ yum install -y open-vm-tools
~$ sudo systemctl enable --now vmtoolsd.service
```
