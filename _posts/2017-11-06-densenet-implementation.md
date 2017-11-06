---
layout: post
mathjax: true
category: 深度学习
title: DenseNet 密集连接卷积网络 TensorFlow 实现
tags: DenseNet 深度学习 分类 识别 TensorFlow
author: Alvin Zhu
date: 2017-11-06
---

* content
{:toc}
最近使用TensorFlow的高级API（Dataset和Estimator）实现了DenseNet，代码地址：[py_densenet](https://github.com/AlvinZhu/py_densenet)。

包括DenseNet和DenseNet-BC，包括CIFAR和ImageNet，测试了CIFAR10，复现了论文效果。ImageNet没测完，训练时间太长了。