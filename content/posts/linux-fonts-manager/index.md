---
title: "Linux下的字体管理"
description: "linux-fonts-manager"
keywords: "linux,fonts"
date: 2023-12-02T02:47:34+08:00
lastmod: 2023-12-02T02:47:34+08:00
tags: [Linux]
categories: [Linux]
---

本文介绍Linux下的字体管理，不同字体族之间的关系，使用Fontconig管理字体，推荐了几款我使用的优秀字体。

<!--more-->

## 三个通用字体族名 (generic family)

- snas-serif 无衬线
- serif 衬线
- monospace 等宽字体

## ttf 和otf 的区别

TTF（TrueType Font）是Apple公司和Microsoft公司共同推出的字体文件格式，随着windows的流行，已经变成最常用的一种字体文件表示方式。 而OTF（OpenType Font）是TTF 的升级版，而OTF 是采用的是PostScript 曲线，支持OpenType 高级特性的更高级字体。

## 字体文件存储目录

- /usr/share/fonts —> 包管理安装
- /usr/local/share/fonts —> 自己手动安装的全局字体
- ~/.local/share/fonts —> 自己手动安装的用户字体

终端中查看字体

```Bash
alias sf='fc-list : family style | fzf'
```

## 中文字体

中文字体推荐谷歌的思源黑体`Noto Sans SC`，华为的鸿蒙黑体`HarmonyOS Sans SC`，小米的`MiSans`

```bash
$ yay -S ttf-harmonyos-sans
```

## Nerd字体

[Nerd字体](https://www.nerdfonts.com/)可以为原来的等宽字体添加图标，icon等符号，多用于终端和编程环境中，丰富显示效果，常用的Nerd字体。

```bash
 yay -Qsq nerd
otf-comicshanns-nerd
otf-commit-mono-nerd
ttf-cascadia-code-nerd
ttf-fantasque-nerd
ttf-firacode-nerd
ttf-inconsolata-nerd
ttf-nerd-fonts-symbols
ttf-nerd-fonts-symbols-common
ttf-roboto-mono-nerd
```

## emoji字体

```Bash
# Twitter推出的emoji字体
$ yay -S ttf-twemoji-color

# Google Noto emoji fonts
$ yay -S extra/noto-fonts-emoji
```

## Fontconfig

Fontconfig可以用来对Linux下的字体进行统一的管理，任何使用 fontconfig 的程序都会遵守这一标准，不用为每个APP单独设置字体，统一使用字体族名就可以了。

```xml
$ nvim ~/.config/fontconfig/fonts.conf
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif SC</family>
      <family>Symbols Nerd Font</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>HarmonyOS Sans SC</family>
      <family>Noto Sans SC</family>
      <family>Symbols Nerd Font</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>

  <alias>
    <family>monospace</family>
    <prefer>
      <family>FiraCode Nerd Font Mono</family>
      <family>Noto Sans Mono</family>
      <family>Noto Color Emoji</family>
      <family>Braille</family>
    </prefer>
  </alias>
</fontconfig>
```


> 经过测试这种方式只在`LANG=en_US.UTF-8`的环境中有效!需要添加环境变量`FC_LANG=und`才能在`LANG=zh_CN.UTF-8`时生效。可写入`/etc/environment`


## Unicode

2023 年每个软件开发者都必须知道的关于 Unicode 的最基本的知识
<https://blog.xinshijiededa.men/unicode/>

## 参考

- [How to Install and Manage Fonts on Linux](https://youtu.be/1RtLyPzbttA)

- [fontconfig的原理](https://catcat.cc/post/2021-03-07/)
