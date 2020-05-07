---
layout: post
title:  "secureCRT通过SFTP命令上传下载文件 "
date:   2020-05-07 00:00:00
categories: 工具
tags: 工具
excerpt: 最近使用服务器调试代码频繁，需要在服务器里上传下载文件,secureCRT中不需要额外下载工具就能实现文件上传下载
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)



> [博客地址](https://dufaxing.com){:target="_blank"}


# SFTP命令


secureCRT的file菜单栏中连接STFP &emsp; session，或者`Alt+p`连接STFP



![Ym0yTA.png](https://s1.ax1x.com/2020/05/07/Ym0yTA.png)


`lpwd`来查看本地的下载路径

`lcd`命令进入需要上传文件的本地路径

`lls`命令查看本地的文件

`put`命令把本地的文件上传到服务器

`get`命令把服务器端文件下载到本地

---
