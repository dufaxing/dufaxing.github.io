---
layout: post
title:  "IAR编译报错"
date:   2017-9-22 00:00:00
categories: 工具
tags: IAR
excerpt: IAR编译报错原因分析以及解决办法
mathjax: true
---
* content
{:toc}
---

## 报错现象
### .o文件中未定义变量

原因：程序文件extern定义的变量在其他文件中找不到定义，检查extern 引用变量、函数的拼写。

---

### IAR编译“长度为变量的数组”时报错  
* 现象

![](http://wx1.sinaimg.cn/mw690/e4439297gy1fjsb1kz955j20f900ugle.jpg)

![](http://wx4.sinaimg.cn/mw690/e4439297gy1fjsb1lhv3dj20fb01st8h.jpg)

* 原因

C99支持数组长度下标为变量，C89不支持，在IAR设置Options->C/C++ Complier下的Language中，选择C99并勾选Allow VLA，即允许变量长度数组Variable length arrays，这样程序中的数组可以使用变量做下标了。

注意：

>VLA只表示数组声明时长度可以是变量，一旦声明完毕数组长度不可变，并不能动态伸缩；
>变长数组不能存放在静态存储区，即不能是全局变量和静态变量；
>变长数组不能在声明时初始化，即int size=3;arr[size] = {0}是错误的，因为数组下标是变量，编译时编译器不知道其确切长度，只有在运行时其长度才确定。


* 解决办法

![](http://wx2.sinaimg.cn/mw690/e4439297gy1fjsb1lxdgqj20g80dxaaq.jpg)

编译通过

![](http://wx2.sinaimg.cn/mw690/e4439297gy1fjsb1mdvg7j20fi01c743.jpg)





---


