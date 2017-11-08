---
layout: post
mathjax: true
category: 开发
title: Markdown教程
tags: Markdown 编码风格 开发
author: Alvin Zhu
date: 2017-10-07
---

* content
{:toc}

## Markdown简介

Markdown是一种轻量级、简单的语法，用于编写文档。使用Markdown可以控制文档的显示。比如：粗体、斜体、图片、列表、表格、段落、代码、引用、公式等。

我们使用的Markdown语法遵循[Github Flavored Markdown](https://github.github.com/gfm/)标准。也可以参阅[Mastering Markdown](https://guides.github.com/features/mastering-markdown/)这个简短的教程。

推荐的Markdown编辑器为[Typora](https://typora.io/)，也可使用其它遵循[Github Flavored Markdown](https://github.github.com/gfm/)标准的编辑器。






## 基本Markdown语法

本节仅对通用的Markdown语法进行简要介绍，完整Markdown语法请参考[Github Flavored Markdown](https://github.github.com/gfm/)标准。

### 标题

```markdown
# 一级标题
## 二级标题
### 三级标题
###### 六级标题
```

实际效果参见本文标题

### 强调

```markdown
*倾斜*
_倾斜_
**加粗**
__加粗__
_倾斜**加粗并倾斜**倾斜_
```

*倾斜*
_倾斜_
**加粗**
__加粗__
_倾斜**加粗并倾斜**倾斜_

### 无序列表

```markdown
* 一
* 二
	* 二一
	* 二二
```

* 一
* 二
  * 二一
  * 二二

### 有序列表


```markdown
1. 一
1. 二
	1. 二一
	1. 二二
```

1. 一
2. 二
   1. 二一
   2. 二二

### 图片

```markdown
![头像](/favicon.ico)
```

![头像](/favicon.ico)

### 链接

```markdown
[Github](http://github.com)
```

[Github](http://github.com)

### 引用

```markdown
本文第一节写道：
> Markdown是一种轻量级、简单的语法，用于编写文档。
> 我们使用的Markdown语法遵循Github Flavored Markdown标准。
```
本文第一节写道：
> Markdown是一种轻量级、简单的语法，用于编写文档。
> 我们使用的Markdown语法遵循Github Flavored Markdown标准。

### 嵌入式代码

```markdown
OpenCV使用`imread`函数读取图片。
```

OpenCV使用`imread`函数读取图片。

## Github扩展的Markdown语法

本节仅对Github扩展的Markdown语法进行简要介绍，完整Markdown语法请参考[Github Flavored Markdown](https://github.github.com/gfm/)标准。

### 代码块与语法高亮

```markdown
​```python
def add(a, b):
	return a + b
​```
```

```python
def add(a, b):
	return a + b
```
### 任务列表

```markdown
- [x] 已完成的任务
- [ ] 未完成的任务
```

- [x] 已完成的任务
- [ ] 未完成的任务

### 表格

```markdown
表头1   | 表头2
---- | ----
单元格1 | 单元格2
单元格3 | 单元格4
```

| 表头1  | 表头2  |
| ---- | ---- |
| 单元格1 | 单元格2 |
| 单元格3 | 单元格4 |

### 自动链接

```markdown
http://www.github.com/
```

http://www.github.com/

### 删除线

```markdown
~~删除~~
```

~~删除~~

### Emoji表情

```markdown
:smile:
:+1:
```

:smile:
:+1:

参见Emoji Cheat Sheet](http://www.emoji-cheat-sheet.com/)查看支持的表情。
