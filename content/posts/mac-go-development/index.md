---
title: Mac OS下设置 Go 语言开发环境
date: 2017-09-09T02:47:34+08:00
categories: [Golang]
tags: [Mac]
---
Go 语言是由`Google`推出的一门编程语言，被认为是21世纪的 C语言，本文介绍如何在 `Mac OS`下设置`Go`语言开发环境，最快捷的方式使用`brew`安装 Go,再设置相应的环境变量。
<!--more-->

### 安装 Go

    $ brew install go

### 设置环境变量
我使用的`Shell`是`zsh`,其它的依次类推，编辑家目录下的`.zshenv`，如果使用的`bash`,则相应的文件为`~/.bash_profile`
```sh
$ vim ~/.zshenv
 # Go development
export GOPATH=$HOME/dev/go
export GOROOT="$(brew --prefix golang)/libexec"
export PATH="$PATH:$GOPATH/bin:$GOROOT/bin"
```

### 使环境变量生效
在命令行终端执行

```shell
$ sourse ~/.zshenv
```

### 查看环境变量是否生效
```sh
$ go env
GOARCH="amd64"
GOBIN="/User/pengshp/dev/go/bin"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/pengshp/dev/go"
GORACE=""
GOROOT="/usr/local/opt/go/libexec"
GOTOOLDIR="/usr/local/opt/go/libexec/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
```

### 开发工具
对于初期的学习老说，开发工具可以选用`Sublime Text 3`或者是`VScode`,Sublime安装一个`GoSublime`的插件，VScode也有相应的插件。

### 参考链接

1、Go官网：<https://golang.google.cn/>

2、Go中文社区：<https://studygolang.com/>

3、Go入门指南：<http://wiki.jikexueyuan.com/project/the-way-to-go/>
