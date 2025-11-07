---
title: "树莓派Archlinux Arm 基本配置教程"
date: 2018-10-02T02:47:34+08:00
categories: [Linux]
tags: [Linux, RaspberryPi]
---

本文主要介绍在树莓派上配置Ａrchlinux系统，因为Ａrchlinux是精简安装，所以安装完后要做一些基本的配置和优化。可根据自己的需要进行配置，这里是我的配置记录，可供参考。

<!--more-->

#### 一.基本配置

##### 配置网络

1. 有线网络

```bash
# 有线网络
~$ sudo vim /etc/systemd/network/20-wired.network
[Match]
Name=eth0  # w
Type=ether

[Network]
DHCP=yes

## 静态IP
[Network]
Address=10.1.10.9/24
Gateway=10.1.10.1
DNS=10.1.10.1
#DNS=8.8.8.8

# 启动网络服务
~$ sudo systemctl enable --now systemd-networkd.service
```

2. 无线网络wlan

```bash
# 安装包
$ pacman -S wireless_tools dialog libnl wpa_supplicant

# SSID PASSWD
~$ wpa_passphrase HUAWEI 12345678
network={
        ssid="HUAWEI"
        #psk="12345678"
        psk=3a0da8cfedc18f6d7d8c61427378d2a9b1348b3541055535cc
}

~$ sudo vim /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=wheel
update_config=1
fast_reauth=1
ap_scan=1
network={
        ssid="HUAWEI"
        #psk="12345678"
        psk=3a0da8cfedc18f6d7d8c61427378d2a9b1348b3541055535cc
}

~$ sudo systemctl enable --now wpa_supplicant@wlan0.service

~$ sudo vim /etc/systemd/network/25-wireless.network
[Match]
Name=wlan0 # 网卡名
Type=wlan

[Network]
DHCP=yes

~$ sudo systemctl enable --now systemd-networkd.service
```

##### 1.更新系统

```bash
sudo pacman -Syu
```

##### 2.更新时区

```bash
timedatectl set-timezone Asia/ShangHai
```

##### 3.把用户加入用户组

```bash
pacman -S sudo
```

nano /etc/sudoers
添加

```bash
# 执行sudo 时不需要输入密码
%wheel ALL=(ALL) NOPASSWD: ALL
```

##### ４.安装中文字体

```bash
# 安装思源黑体字体
~$ sudo pacman -S adobe-source-han-sans-cn-fonts
```

##### ５.基本开发环境

```bash
~$ sudo pacman -Sy  git vim
~$ sudo pacman -Sy tldr
~$ sudo pacman -Sy bat tree
```

##### ６.安装Python3

```bash
sudo pacman -S python python-pip ipython
```

##### ７.系统监测

```bash
sudo pacman -S htop neofetch
```

#### 二.安装桌面环境

##### 1.安装基本组建

```bash
sudo pacman -S base-devel
```

##### 2.安装桌面服务

```bash
sudo pacman -S xorg-xinit xorg-server xorg-utils xorg-server-utils
```

##### 3.安装音视频相关驱动

```bash
sudo pacman -S xf86-video-fbdev xf86-video-vesa
```

##### 4.安装mate桌面环境

```bash
sudo pacman -S mate mate-extra
```

##### 5.安装管理器，启动器

```bash
pacman -S openbox lxde gamin dbus mesa
```

##### 6.修改配置文件

添加如下内容

```bash
$ sudo vim ~/.xinitrc
exec mate-session
```

##### 7.关机重启

开机启动后登陆输入**startx**便可以进入mate桌面环境。到此桌面环境配置完成了。

#### 三.进阶配置

##### 1.安装fcitx5输入法

```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons
```

编辑`~/.xinitrc`

```bash
$ vim ~/.xinitrc
export LANG=zh_CN.UTF-8
export XIM=fcitx
export XMODIFIERS="@im=fcitx"
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XIM_PROGRAM=fcitx
fcitx &
```

保存退出！

##### 2.安装火狐浏览器

```bash
$ sudo pacman -S firefox firefox-i18n-zh-cn
```

##### 3.设置中文环境

```bash
$ sudo vim /etc/locale.conf
LANG=zh_CN.UTF-8

# or
~$ localectl set-locale LANG=en_US.UTF-8
~$ localectl status
```

##### 4.音频设置

```bash
$ pacman -Sy alsa-utils alsa-firmware alsa-lib alsa-plugins
```

选择一个音频输出
amixer cset numid=3 x

参数设置为：

> - 0 自动
> - 1 模拟输出
> - 3 HDMI

##### 5.安装zsh

```bash
~$ sudo pacman -Sy zsh
~$ sudo pacman -Sy community/zsh-completions
~$ sudo pacman -Sy community/zsh-autosuggestions
~$ sudo pacman -Sy community/zsh-syntax-highlighting
~$ sudo pacman -Sy community/zsh-theme-powerlevel10k

# 设置alarm的默认shell为zsh
$ chsh -s /bin/zsh alarm
```

#### 6. 使用yay管理 AUR

1. yay简介

[Jguer/yay: Yet another Yogurt - An AUR Helper written in Go (github.com)](https://github.com/Jguer/yay)

yay使用go语言写的`yogurt`的替代品，用来管理AUR包,使用时不需要加`sudo`

```bash
# install yay
pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

2. 基本使用

```bash
$ yay -Sy [pkg]
# 打印系统包信息
$ yay -Ps
# 清除未使用的依赖
$ yay -Yc

# install Google Noto font
~$ yay -Sy noto-fonts-sc
# install OSX fonts
~$ yay -Sy ttf-mac-fonts
```

#### 7. 安装中文man_page

```sh
~$ sudo pacman -Sy community/man-pages-zh_cn
```
