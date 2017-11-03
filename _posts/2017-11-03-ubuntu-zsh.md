---
layout: post
mathjax: true
category: Linux
title: Ubuntu16.04 Zsh配置
tags: Ubuntu Zsh Linux
author: Alvin Zhu
date: 2017-11-03
---

* content
{:toc}
Zsh功能强大，Zsh是我的默认Shell，并使用[Oh My ZSH](http://ohmyz.sh/)和[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)插件。

但是当我使用ssh远程连接我的电脑时，Zsh没有引用`.profile`文件。仔细检查后，也没有引用`/etc/profile`等与**login shell**相关的配置文件，导致了我的`PATH`等环境变量配置不正确。怎么解决呢？





## 解决方法

1. 修改`/etc/zsh/zprofile`文件，在文件末尾添加一行（这个应该默认就有的，不知道为什么Ubuntu没加）：

   ```shell
   source /etc/profile
   ```

2. 创建`~/.zprofile`文件，添加如下内容：

   ```shell
   source ~/.profile
   ```

3. 创建`~/.zlogout`文件，添加如下内容：

   ```shell
   source ~/.bash_logout
   ```

这就可以完美解决了。参考了[ArchWiki Zsh](https://wiki.archlinux.org/index.php/zsh)和[ArchWiki Bash](https://wiki.archlinux.org/index.php/bash)。

以下是懒人版：

```shell
echo "source /etc/profile" | sudo tee --append /etc/zsh/zprofile
echo "source ~/.profile" >> ~/.zprofile
echo "source ~/.bash_logout" >> ~/.zlogout
```
