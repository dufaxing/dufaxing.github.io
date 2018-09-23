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


在查找资料过后，在《跟我一起写Makefile》中这样描述：
> make会在当前目录下找到名字为`Makefile`或者`makefile`的文件。如果找到，它会找文件中的第一个目标文件，并把这个文件作为最终的目标文件。由于make的依赖性，make会一层一层的去找依赖关系，直到编译出第一个目标文件。
