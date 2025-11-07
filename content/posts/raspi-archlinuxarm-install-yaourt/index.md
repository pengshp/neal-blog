---
title: 树莓派 Archlinux ARM 安装yaourt
date: 2016-08-29T02:47:34+08:00
categories: [RaspberryPi]
tags: [RaspberryPi,Linux]
---

摘要：`yaourt`是一个比`pacman`更好用的包管理工具，集成了很多额外的软件包。正常情况下树莓派Archlinux ARM不能使用`sudo pacman -S yaourt`安装`yaourt`,下面介绍一种方法安装`yaourt`.

<!-- more -->

### 一.配置环境
#### 1.添加源
修改`/etc/pacman.conf`,添加如下内容
```bash
[archlinuxfr]
Server = http://repo.archlinux.fr/arm
SigLevel = Never
```
#### 2.手动安装package-query
```bash
sudo pacman -S yajl    #安装依赖
curl -O https://aur.archlinux.org/cgit/aur.git/snapshot/package-query.tar.gz
tar -xzvf package-query.tar.gz
cd package-query
makepkg -si
```
如果一切正常，便可以编译生成`package-query-1.8-2-armv7h.pkg.tar.xz`,期间会下载`package-query-1.8.tar.gz`,但是我的一直不能下载下来，显示超时，可以到[这里](http://mir.archlinux.fr/releases/package-query/package-query-1.8.tar.gz)下载,`scp`放到`package-query`目录下。
### 二.安装使用
#### 1.执行下面的命令便可以安装`yaourt`
```bash
cd ..
sudo pacman -S yaourt
```
#### 2.基本使用
```bash
yaourt -S package_name – 从AUR安装软件包
yaourt -Ss password – 使用关键字搜索软件包
yaourt -Syu –aur – 从AUR升级本地软件数据库并安装更新
yaourt -Si package_name – 列出软件包信息
yaourt -Sc – 从缓存中清楚旧的软件包
yaourt -Su – 安装AUR中的更新软件包
yaourt -Sy – 获取最新的AUR软件包数据库
yaourt -Cd – 清楚AUR软件包数据库
yaourt -R package_name – 删除软件包
```
> 到此`yaourt`的安装介绍就完了，体验一下`yaourt`的强大吧！enjoy it!

> 更新：yaourt已被弃用，推荐使用Go语言编写的yay,功能更加强大！

[Jguer/yay: Yet another Yogurt - An AUR Helper written in Go (github.com)](https://github.com/Jguer/yay)

