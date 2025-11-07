---
title: "Arch Linux设置rofi"
date: 2022-04-25T20:37:46+08:00
tags: [linux]
categories: [linux]
---

[rofi](https://github.com/lbonn/rofi) : A window switcher, run dialog and dmenu replacement. 简单说就相当于我们使用的`utools`, 可以启动应用，搜索文件，搜索emoji,执行命令等。但utools在Arch Linux的KDE Plasma桌面环境的兼容性不好，于是找到了替代品rofi.试了一下，效果很不错，分享一下自己的配置。

<!--more-->

我所使用的窗口管理器是`wayland`,所以安装该环境下的兼容版本。

Rofi支持下面的模式（Modes）

- **run**: launch applications from $PATH, with option to launch in terminal.
- **drun**: launch applications based on desktop files. It tries to be compliant to the XDG standard.
- **window**: Switch between windows on an EWMH compatible window manager.
- **ssh**: Connect to a remote host via ssh.
- **filebrowser**: A basic file-browser for opening files.
- **keys**: list internal keybindings.
- **script**: Write (limited) custom mode using simple scripts.
- **combi**: Combine multiple modes into one.

可以按 `Shift+左右方向键`切换不同的Modes.

### 1. 安装

```shell
# 或者使用yay安装
~$ sudo pacman -S rofi

# emoji搜索支持
~$ sudo pacman -S community/rofi-emoji
```

要把emoji复制到剪切板，需要安装

```shell
~$ sudo pacman -S wl-clipboard
```

### 2. 配置

```shell
~$ mkdir -p ~/.config/rofi

# 生成默认配置
~$ rofi -dump-config > ~/.config/rofi/config.rasi
```

由于我的KDE桌面的主题是Nordic,所以rofi也安装这个主题，比较搭配。安装[Nordic主题](https://github.com/undiabler/nord-rofi-theme)，下载`nord.rasi`文件放到`~/.config/rofi/config.rasi`下，修改配置文件引用该主题，也可添加自定义的设置,下面是我的配置

```shell
~$ vim ~/.config/rofi/config.rasi
/** Basic config file **/

configuration {
    modes: [combi];
    modi: "drun,run,filebrowser,emoji,ssh,combi";
    font: "MiSans 20";
    width: 24;
    display-ssh:    "";
    display-emoji:    "";
    display-filebrowser:    "";
    display-run:    "";
    display-drun:   "";
    drun-display-format: "{icon} {name}";
    display-window: "";
    display-combi:  "";
    show-icons: true;
    //icon-theme: "Papirus";
    icon-theme: "Tela circle purple dark";
}

//setup theme
@theme "nord"
```

### 3. 设置全局快捷键

在系统*设置–> 快捷键 –> 自定义快捷键* 设置启动rofi的快捷键，我设置的是`Alt+Return`

### 4. 效果展示

![Rofi](https://hugoblog-img-1251694304.cos.ap-guangzhou.myqcloud.com/blog/2022-04-25_21-09.png "Rofi")

### 4. 参考

1. <https://github.com/lbonn/rofi>
2. <https://github.com/undiabler/nord-rofi-theme>
