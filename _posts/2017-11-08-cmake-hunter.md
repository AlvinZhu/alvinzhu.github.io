---
layout: post
mathjax: true
category: 开发
title: CMake项目依赖管理组件Hunter
tags: CMake Hunter 开发
author: Alvin Zhu
date: 2017-11-08
---

* content
{:toc}

以前，我编译CMake项目是这样的：

```sh
# 安装依赖
sudo apt-get install foo boo
# 或者Ubuntu源里面没有，直接下载源码编译，然后cmake -D指定依赖项路径。
# 编译
mkdir build
cd build
cmake ..
make -j9
```

现在，使用[Hunter](https://github.com/ruslo/hunter)管理依赖后，是这样的：

```sh
# 依赖是什么？不关心，直接编译
mkdir build
cd build
cmake ..
make -j9
```

CMake默认处理依赖的方式是去系统里面找，找不到就报错。

Hunter处理依赖的方法简单暴力，你的项目依赖什么，直接下源码编译，不管系统里面有没有。

好处当然是更好的跨平台，编译更简单。缺点嘛，编译时间长点。

