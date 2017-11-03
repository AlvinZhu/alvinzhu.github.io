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

但是，经过长期的使用（从8.04开始，Ubuntu就是我的的主系统了），我发现在调用CUDA训练模型的时候，或者其它仅使用NVIDIA的显卡驱动的OpenGL加速的时候，Ubuntu的GUI界面会卡顿或直接死掉。如何解决这个问题让系统更稳定呢？





## 具体方法

怎么配置CUDA环境，或者怎么装NVIDIA的私有驱动，我就不再赘述了，可以网上搜下。为了解决上述问题。我只介绍一下我的配置的不同点：

区别只有一点，安装NVIDIA私有驱动，这里不装Ubuntu源里面的驱动，而是使用NVIDIA官网下载的`.run`文件安装驱动，安装的时候使用以下参数：

```sh
sh ./NVIDIA-*.run --no-opengl-files
```

这样安装的驱动不包含OpenGL部分，也就是不使用NVIDIA的驱动去加速Ubuntu的GUI界面和其它3D应用。OpenGL这部分就有劳Intel的核显了。（你要是还要用Ubuntu玩游戏，那请忽略此文，何必呢，装个Windows双系统呗，或者KVM+GPU Pass through，至于KVM怎么弄，有时间再详细写）。

这样，用核显做显示，性能很够用了，而且驱动很稳定，GUI再也没死过了。AMD的显卡驱动也还不错，插一个低端AMD卡专门做显示也是可以的。NVIDIA的私有驱动就专注于CUDA吧。

后面安装CUDA、CUDNN也都从NVIDIA官网下载`.run`文件和`.tar.gz`文件，就不要装`.deb`包了，依赖不满足，不折腾。

