---
layout: post
mathjax: true
category: Linux
title: Ubuntu 开启 TCP BBR
tags: Ubuntu TCP BBR Linux
author: Alvin Zhu
date: 2017-10-27
---

* content
{:toc}

BBR (Bottleneck Bandwidth and Round-trip) 目的是要充分利用带宽，并且尽量降低网络链路上的 buffer 占用率，从而降低延迟。在有一定丢包率的网络链路上使用 TCP BBR 有着提高传输速度的作用。

Ubuntu16.04，内核版本4.9以上，输入以下命令即可启用：

```sh
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```







