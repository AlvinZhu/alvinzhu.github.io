---
layout: post
mathjax: true
category: Linux
title: Ubuntu SSH 后台执行程序
tags: Ubuntu SSH Linux
author: Alvin Zhu
date: 2017-11-03
---

* content
{:toc}

通过远程ssh启动程序时，后台运行程序并将标准输出重定向到某个日志文件:

```shell
sudo apt install expect
nohup unbuffer python test.py > ./log_test 2>&1 &
```






