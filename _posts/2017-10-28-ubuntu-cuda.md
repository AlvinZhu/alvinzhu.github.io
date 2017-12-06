---
layout: post
mathjax: true
category: Linux
title: Ubuntu16.04 CUDA与OpenGL分离的环境配置
tags: Ubuntu CUDA NVIDIA 驱动 Linux
author: Alvin Zhu
date: 2017-10-28
---

* content
{:toc}

现在深度学习这么火，不可避免的要使用CUDA。在Ubuntu系统中配置CUDA环境的教程也是一大堆。

但是，经过长期的使用，我发现在调用CUDA训练模型的时候，或者其它仅使用NVIDIA的显卡驱动的OpenGL加速的时候，Ubuntu的GUI界面会卡顿或直接死掉。如何解决这个问题让系统更稳定呢？





## 具体方法

怎么配置CUDA环境，或者怎么装NVIDIA的私有驱动，我就不再赘述了，可以网上搜下。为了解决上述问题。我只介绍一下我的配置的不同点：

区别只有一点，安装NVIDIA私有驱动，这里不装Ubuntu源里面的驱动，而是使用NVIDIA官网下载的`.run`文件安装驱动，安装的时候使用以下参数：

```sh
sh ./NVIDIA-*.run --no-opengl-files
```

这样安装的驱动不包含OpenGL部分，也就是不使用NVIDIA的驱动去加速Ubuntu的GUI界面和其它3D应用。OpenGL这部分就有劳Intel的核显了。（你要是还要用Ubuntu玩游戏，那请忽略此文，何必呢，装个Windows双系统呗，或者KVM+GPU Pass through，至于KVM怎么弄，有时间再详细写）。

这样，用核显做显示，性能很够用了，而且驱动很稳定，GUI再也没死过了。AMD的显卡驱动也还不错，插一个低端AMD卡专门做显示也是可以的。NVIDIA的私有驱动就专注于CUDA吧。

后面安装CUDA、CUDNN也都从NVIDIA官网下载`.run`文件和`.tar.gz`文件，就不要装`.deb`包了，依赖不满足，不折腾。

## 更新

还有一种方法：

安装Ubuntu源里面的NVIDIA驱动：

```sh
sudo apt install nvidia-384
```

然后打开`nvidia-settings`，更改`PRIME Profiles`为`intel`，不要管什么省电，笔记本也许可以，台式机仅能切换，不支持省电。

也可以用命令切换：

```sh
sudo prime-select intel
```

注销再登陆进来。这时候已经是用核显显示了。

安装`bumblebee`：

```sh
sudo add-apt-repository ppa:bumblebee/testing
sudo apt update
sudo apt install bumblebee
```

需要确认一些配置：

`/etc/modprobe.d/bumblebee.conf`里面有屏蔽对应版本的驱动，比如我装的是`nvidia-384`，需要有：

```sh
# 384
blacklist nvidia-384
blacklist nvidia-384-updates
blacklist nvidia-experimental-384
```

`/etc/bumblebee/bumblebee.conf`修改一些配置：

```sh
Driver=nvidia
KernelDriver=nvidia-384
LibraryPath=/usr/lib/nvidia-384:/usr/lib32/nvidia-384
XorgModulePath=/usr/lib/nvidia-384/xorg,/usr/lib/xorg/modules
```

`/etc/bumblebee/xorg.conf.nvidia`，去掉`BusID`的注释：

```sh
    BusID "PCI:01:00:0"
    Option "Coolbits" "28"
```

使用`lspci | egrep 'VGA|3D'`查看显卡的`Bus ID`，确保设置正确。

可以添加`Option "Coolbits" "28"`，开启`nvidia-settings`里面的超频和风扇调节。

配置完成，重启。

另外，要打开`nvidia-settings`，需要这样：

```sh
optirun -b none /usr/bin/nvidia-settings  -c :8
```

以上是桌面PC的配置，如果是服务器，还要注意，一般来说，服务器是不启动X的，所以也不需要什么核显显示，独显CUDA，直接`sudo prime-select nvidia`就可以了。

不启动X，无法使用`nvidia-settings`，两种方法解决：

把X启动起来，不用就是了。然后参考以下命令：

```sh
# 启动X
sudo systemctl start graphical.target
# 设置环境变量
export DISPLAY=:0
export XAUTHORITY=/var/run/lightdm/root/:0
# 创建xorg.conf
nvidia-xconfig --allow-empty-initial-configuration --enable-all-gpus --cool-bits=28 --separate-x-screens
# 风扇转速锁定到75%
sudo nvidia-settings -a "[gpu:0]/GPUFanControlState=1" -a "[fan:0]/GPUTargetFanSpeed=75" 
```

另外一种方法，使用这个脚本：

[set_gpu_fans_public](https://github.com/boris-dimitrov/set_gpu_fans_public)