+++
title = 'Archlinux Fcitx5 Rime'
date = '2025-10-27T09:49:05+08:00'
tags = ['linux']
categories = ['linux']
description = ''
+++

## 简介

- Fcitx5 是一款轻量、高效、模块化的 输入法框架（Input Method Framework），主要用于 Linux 系统，支持多语言输入（如中文、日文、韩文等），以高可定制性、低资源占用和良好的兼容性著称。它是 Fcitx（Free Chinese Input Toy for X）项目的新一代版本，重构了架构并引入了更多现代特性。
- RIME（中州韵）不是“输入法”，而是一套开源的“输入法引擎”。它位于操作系统和输入法皮肤之间，负责把键盘按键变成汉字，而长什么样、怎么打、打什么，全由用户自己定。核心特点一句话：高度可定制、跨平台、算法先进、配置即代码。

RIME组成结构

- 引擎核心：librime（C++）
- 前端（不同平台）：
  1. Windows：Weasel（小狼毫）
  2. macOS：Squirrel（鼠须管）
  3. Linux：Fcitx5-rime / IBus-rime
  4. Android：Fcitx5-Android
  5. iOS：Hamster(仓输入法)
- 数据包：schema（方案）+ dict（词典）+ patch（补丁）

## 安装

```bash
sudo pacman -S fcitx5-im fcitx5-rime
```

fcitx5-im是组包，包含下面四个软件包

- fcitx5
- fcitx5-configtool
- fcitx5-qt
- fcitx5-gtk

## 配置

我使用主流的薄荷输入法方案[oh-my-rime](https://www.mintimate.cc/zh/)

### 安装方案

```bash
$ git clone --depth=1 https://github.com/Mintimate/oh-my-rime.git ~/.local/share/fcitx5/rime
```

在安装后，使用『Ctrl』+『~』进行切换。（默认激活的是『薄荷拼音-全拼输入』）。我平时主要使用薄荷全拼和小鹤双拼。
后面更新可以使用`oh-my-rime-cli`工具[link](https://github.com/Mintimate/oh-my-rime-cli)

### 配置方案

```yaml
$ vim ~/.local/share/fcitx5/rime/default.custom.yaml
patch:
  schema_list:
    - schema: rime_mint # 薄荷拼音
    - schema: double_pinyin_flypy # 小鹤双拼
  # 启用Shift上屏英文并进行中英文切换
  "ascii_composer/switch_key/Shift_L": commit_code
  app_options:
    org.wezfurlong.wezterm:
      ascii_mode: true
    Alacritty:
      ascii_mode: true
    com.mitchellh.ghostty:
      ascii_mode: true
```

### 配置薄荷拼音

```yaml
$ vim ~/.local/share/fcitx5/rime/rime_mint.custom.yaml
patch:
  # 语言模型
  "grammar/language": wanxiang-lts-zh-hans
  "grammar/collocation_max_length": 8
  "grammar/collocation_min_length": 2
  # translator 内加载
  "translator/contextual_suggestions": false
  "translator/max_homophones": 5
  "translator/max_homographs": 5
  "style/horizontal": true
  "menu/page_size": 6
  # 自定义短语
  "engine/translators/+":
    - table_translator@custom_phrase
  custom_phrase:
    dictionary: ""
    user_dict: custom_phrase # 需要手动创建 custom_phrase.txt 文件
    db_class: stabledb
    enable_completion: true # 补全提示
    enable_sentence: false # 禁止造句
    initial_quality: 99 # custom_phrase 的权重应该比其他词库大
```

配合万象拼音的语言模型，可以实现长难句的准确输入。下载万象语言模型到rime目录

```bash
$ cd ~/.local/share/fcitx5/rime
$ wget https://github.com/amzxyz/RIME-LMDG/releases/download/LTS/wanxiang-lts-zh-hans.gram
```

### 皮肤

皮肤可以使用`catppuccin`制作的fcitx5皮肤[catppuccin-fcitx5](https://github.com/catppuccin/fcitx5)

## 参考

- [oh-my-rime](https://www.mintimate.cc/zh/)
- [RIME-LMDG](https://github.com/amzxyz/RIME-LMDG)
- [Rime](https://github.com/rime/librime)
