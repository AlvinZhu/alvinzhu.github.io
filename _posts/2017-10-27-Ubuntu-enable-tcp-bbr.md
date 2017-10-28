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

内核版本4.9以上，输入以下命令即可：

```sh
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```







