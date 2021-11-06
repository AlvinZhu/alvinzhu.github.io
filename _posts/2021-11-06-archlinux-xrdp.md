---
layout: post
mathjax: true
category: Linux
title: 配置Xrdp - 通过RDP连接Arch Linux KDE远程桌面
tags: Arch Linux Xrdp RDP 远程桌面 KDE
author: Alvin Zhu
date: 2021-11-6
---

* content
{:toc}
Windows的RDP远程桌面好用，清晰、低延迟、网络流量低、剪贴板文件同步。常年用Linux的我偶尔需要用一下Windows的时候，就用`Remmina`连一下我的另外一台Windows电脑。但是反过来呢？当我想连一下我的Arch Linux的时候该怎么办呢？`ssh`？可以解决大多数需求。`ssh -X`配合MobaXterm，将就能用，很慢很卡，再说iPad也没有MobaXterm。或者用`TeamViewer`，时不时的给你断一下（内网直连也不是很稳定），烦。

以前我也尝试过Linux下的VNC和RDP方案，问题太多基本没法用。最近又尝试了下，好用！






最早用Ubuntu 8.04，简单，开箱即用。后来折腾了一段时间Gentoo，可定制性极强，系统的方方面面都自己掌控，就是每次编译时间有点长。工作以后没时间折腾，又换回了Ubuntu。什么？Windows？一边玩去，很久很久以前确实挺喜欢折腾Windows的，但是自从用了Linux，再也不想碰Windows。再后来，实在忍受不了Ubuntu自作聪明的GPU Manager和对上游软件包的乱改，加入了Arch邪教。不知不觉Arch Linux已经用了3年，再也没有重装过系统，随着自己对Arch Linux越來越熟悉，配置的越來越完善，可以养老了。最近又补足了一个短板，远程桌面。

