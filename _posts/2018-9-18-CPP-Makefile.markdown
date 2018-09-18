---
layout: post
title:  "Makefile学习与记录"
date:   2018-9-19 00:00:00
categories: C++
tags: Makefile
excerpt: Makefile语法
mathjax: true
---
* content
{:toc}

### Makefile学习遇到的第一个问题

- 在写第一个Makefile的时候遇到如下问题，`sum`如果不写在第一行，不能生成可执行文件。如下图所示：<br/>

![](http://owlypioka.bkt.clouddn.com/makefile_1.png)


- 将'sum'写在Makefile的第一行，可以正确输出`du`可执行文件。如下图所示：<br/>

![](http://owlypioka.bkt.clouddn.com/makefile_2.png)



