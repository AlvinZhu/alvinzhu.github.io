---
layout: post
mathjax: true
category: Linux
title: CUDA 指定GPU
tags: Ubuntu CUDA GPU Linux
author: Alvin Zhu
date: 2017-11-03
---

* content
{:toc}

在多GPU的机器上，指定使用某（几）个GPU运行CUDA相关的程序：

```sh
CUDA_VISIBLE_DEVICES=0,1 python train_cnn.py
```









