---
layout: post
mathjax: true
category: Linux
title: Ubuntu常用软件与配置
tags: Ubuntu Applications Linux
author: Alvin Zhu
date: 2017-11-07
---

* content
{:toc}

本文总结下Ubuntu上常用的软件与必要的配置。

## 常用软件

- Chrome：最常用的浏览器
- VMware Workstation：虚拟机
- JetBrains Toolbox：各种JetBrains的IDE
- Zeal：开源版本的Dash，各自离线API文档查询
- Youdao Dict：有道词典，只能取词，划译是废的。另外也可以用py-googletrans，翻译质量更好更方便。
- Wine：运行Windows软件，exe文件会显示出图标和版本号，也有商业版的crossover。
- Vim：编辑器，非图形界面下用。
- VLC：视频播放器
- VS Code：编辑器，比gedit功能强大，但是比JetBrains的IDE又要差一些，比较尴尬，用的不多。
- Typora：Markdown编辑器
- indicator-multiload：System Monitor的Top Panel插件
- synergy：一套键鼠控制多台电脑
- synaptic：图形界面的软件包管理器
- Qt creater：Qt的IDE，也可做一般的C++ IDE使用。
- psensor：监控CPU等温度的插件
- openjdk：Java JDK
- cuda：NVIDIA计算平台
- netease cloud music：网易云音乐
- meld：图形界面的diff工具
- fcitx sunpinyin：拼音输入法，还要装上cloudpinyin。
- htop：增强版top
- gparted：分区工具
- gitkraken：图形界面的git工具
- gimp：图像处理软件
- gedbi：deb包图形安装界面
- cmake-gui：cmake的GUI
- indicator-cpufreq：查看和控制CPU频率
- paper icon theme：图标
- numix：主题
- Teamviewer：远程桌面
- proxychains：命令行代理

## 必要的配置

**更新源**

apt源用阿里云的

python源用pmm来配置，清华或者豆瓣的，阿里云有问题。

**输入法**

首先打开”System Settings“，打开”Language Support“，让它自动安装下缺失的语言包组件。然后：

```sh
sudo apt install fcitx fcitx-sunpinyin fcitx-module-cloudpinyin
sudo apt-get purge fcitx-moudle-kimpanel
```

**主题**

numix主题和paper的图标，使用unity tweak tool来设置主题

**grub**

```sh
GRUB_DISABLE_OS_PROBER=true

GRUB_DEFAULT=0
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=3
GRUB_TIMEOUT_STYLE=countdown

GRUB_CMDLINE_LINUX_DEFAULT="quiet pci=noaer"
```

**top panel插件**

```sh
sudo apt install indicator-mutiload indicator-cpufreq psensor
```

**lightdm**

关闭guest登陆

**字体**

Noto字体

修改`etc/fonts/conf.avail/64-language-selector-prefer.conf`，把SC放在前面。



