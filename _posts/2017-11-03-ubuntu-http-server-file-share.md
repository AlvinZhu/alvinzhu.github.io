---
layout: post
mathjax: true
category: Linux
title: Ubuntu 使用Python HTTP Server共享文件
tags: Ubuntu Python HTTP Server Share Linux
author: Alvin Zhu
date: 2017-11-03
---

* content
{:toc}

在需要共享文件的目录下，终端执行：

```shell
#Python2
python2 -m SimpleHTTPServer
#Python3
python3 -m http.server
```

默认使用8000端口，需要修改端口，就这样：

```shell
#Python2
python2 -m SimpleHTTPServer 8080
#Python3
python3 -m http.server 8088
```

参见[Python2](https://docs.python.org/2/library/simplehttpserver.html)和[Python3](https://docs.python.org/3/library/http.server.html)文档







