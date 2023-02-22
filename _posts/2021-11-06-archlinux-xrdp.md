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

截图是Remmina连接

![Remmina截图](/assets/2021-11-06-archlinux-xrdp/remmina.png)

配置过程意想不到的简单。基本上参考[ArchWiki](https://wiki.archlinux.org/title/xrdp)

```bash
# 安装必要的软件包，其中xorgxrdp-glamor可以硬件加速
yay -S xrdp xorgxrdp-glamor pulseaudio-module-xrdp
# 修改配置文件
echo "allowed_users=anybody" | sudo tee /etc/X11/Xwrapper.config
cp /etc/X11/xinit/xinitrc ~/.xinitrc
# 注释掉.xinitrc最后几行
# twm &
# xclock -geometry 50x50-1+1 &
# xterm -geometry 80x50+494+51 &
# xterm -geometry 80x20+494-0 &
# exec xterm -geometry 80x66+0+0 -name login
echo "xport DESKTOP_SESSION=plasma" >> ~/.xinitrc
echo "exec /usr/lib/plasma-dbus-run-session-if-needed startplasma-x11"  >> ~/.xinitrc

# /etc/pam.d/xrdp-sesman 加入kwallet5相关的配置
-auth   optional  pam_kwallet5.so
session         optional        pam_keyinit.so force revoke
-session  optional  pam_kwallet5.so auto_start

# /etc/polkit-1/rules.d/49-nopasswd_global.rules 需要添加一些配置
/* Allow members of the wheel group to execute any actions
 * without password authentication, similar to "sudo NOPASSWD:"
 */ 
polkit.addRule(function(action, subject) {
    if ((  action.id == "org.freedesktop.policykit.exec"
    || action.id == "org.fedoraproject.FirewallD1.all"
    || action.id == "org.fedoraproject.FirewallD1.config"
    || action.id == "org.freedesktop.NetworkManager.settings.modify.system") &&
      subject.isInGroup("wheel")) {
        return polkit.Result.YES;
    }
});
# 如果还有权限问题
sudo systemctl edit polkit.service
# 打开调试
[Service]
Environment=G_MESSAGES_DEBUG=all
ExecStart=
ExecStart=/usr/lib/polkit-1/polkitd
# 然后把相应的权限添加到rules中

# 启用xrdp服务
sudo systemctl enable --now xray.service xrdp-sesman.service
```

然后就拿出iPad，用微软的RDP客户端连Arch Linux吧。微软的RDP客户端有个问题，不能手动设置`Network connection type`。需要设置成`LAN`来开启压缩，不然网络流量会很惊人。iPad上默认是对的，但是调高分辨率后，反而没有开启压缩了。

iPad+键盘，连远程桌面写代码，这才是生产力！

