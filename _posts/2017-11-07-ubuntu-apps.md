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

- [Chrome](https://www.google.com/chrome/browser/desktop/index.html) 最常用的浏览器
- [VMware Workstation](https://www.vmware.com/products/workstation-pro.html) 虚拟机
- [JetBrains Toolbox](https://www.jetbrains.com/toolbox/) 各种JetBrains的IDE
- [Youdao Dict](http://cidian.youdao.com/index-linux.html) 有道词典，只能取词，划译是废的。另外也可以用[py-googletrans](https://github.com/AlvinZhu/py-googletrans)，翻译质量更好更方便。
- [VS Code](https://code.visualstudio.com/) 编辑器，比gedit功能强大，但是比JetBrains的IDE又要差一些，比较尴尬，用的不多。
- [Typora](https://typora.io/) Markdown编辑器
- [synergy](https://symless.com/synergy) 一套键鼠控制多台电脑
- [netease cloud music](https://music.163.com/) 网易云音乐
- [GitKraken](https://www.gitkraken.com/) 图形界面的git工具
- [Teamviewer](https://www.teamviewer.com) 远程桌面

## 必要的配置

**更新源**

apt源用阿里云的(`/etc/apt/sources.list`):

```sh
# deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted

# deb http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted
# deb http://security.ubuntu.com/ubuntu xenial-security main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu 
## team, and may not be under a free licence. Please satisfy yourself as to 
## your rights to use the software. Also, please note that software in 
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu xenial partner
# deb-src http://archive.canonical.com/ubuntu xenial partner

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
# deb-src http://security.ubuntu.com/ubuntu xenial-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
# deb-src http://security.ubuntu.com/ubuntu xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
# deb-src http://security.ubuntu.com/ubuntu xenial-security multiverse
```

python源用pmm来配置，清华或者豆瓣的，阿里云有问题:

```sh
pip install pmm
pmm
```



**APT里面的常用工具**

- Wine：运行Windows软件，exe文件会显示出图标和版本号，也有商业版的crossover。
- Vim：编辑器，非图形界面下用。
- VLC：视频播放器
- synaptic：图形界面的软件包管理器
- openjdk：Java JDK
- meld：图形界面的diff工具
- htop：增强版top
- gparted：分区工具
- gimp：图像处理软件
- gedbi：deb包图形安装界面
- cmake-gui：cmake的GUI
- proxychains：命令行代理
- Zeal：开源版本的Dash，各自离线API文档查询

```sh
sudo apt install proxychains cmake-qt-gui gedbi gimp gparted htop meld default-jdk synaptic vim vlc wine zeal
```



**输入法**

首先打开”System Settings“，打开”Language Support“，让它自动安装下缺失的语言包组件。然后：

```sh
sudo apt install fcitx fcitx-sunpinyin fcitx-module-cloudpinyin
sudo apt-get purge fcitx-moudle-kimpanel
```

并设置输入法为fcitx，sunpinyin。



**主题**

numix主题和paper的图标，使用unity tweak tool来设置主题：

```sh
sudo add-apt-repository ppa:numix/ppa
sudo add-apt-repository ppa:snwh/pulp
sudo apt update
sudo apt install numix-gtk-theme paper-icon-theme unity-tweak-tool
```



**grub**

修改`/etc/default/grub`文件，设置为倒数3秒、关闭其它系统的检测、以及去除一些pci的报错。

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

分别用于监控CPU内存网络磁盘使用、频率、温度等。

```sh
sudo apt install indicator-mutiload indicator-cpufreq psensor
```



**lightdm**

关闭guest登陆：

```sh
sudo sh -c 'printf "[Seat:*]\nallow-guest=false\n" >/etc/lightdm/lightdm.conf.d/50-no-guest.conf'
```



**字体**

Noto字体

修改`/etc/fonts/conf.avail/64-language-selector-prefer.conf`，把SC放在前面：

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
	<alias>
		<family>sans-serif</family>
		<prefer>
			<family>Noto Sans CJK SC</family>
			<family>Noto Sans CJK TC</family>
			<family>Noto Sans CJK JP</family>
		</prefer>
	</alias>
	<alias>
		<family>serif</family>
		<prefer>
			<family>Noto Serif CJK SC</family>
			<family>Noto Serif CJK TC</family>
			<family>Noto Serif CJK JP</family>
		</prefer>
	</alias>
	<alias>
		<family>monospace</family>
		<prefer>
			<family>Noto Sans Mono CJK SC</family>
			<family>Noto Sans Mono CJK TC</family>
			<family>Noto Sans Mono CJK JP</family>
		</prefer>
	</alias>
</fontconfig>
```