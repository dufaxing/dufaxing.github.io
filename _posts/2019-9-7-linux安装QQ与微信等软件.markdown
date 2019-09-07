---
layout: post
title:  "NULL"
date:   2019-9-7 00:00:00
categories: linux
tags: 工具
excerpt: 在Ubuntu中安装TIM与微信等Windows工具
mathjax: true
---
* content
{:toc}
---


> [博客地址](https://dufaxing.com){:target="_blank"}


### 第1步:安装deepin-wine环境

安装deepin-wine环境：上[https://github.com/wszqkzqk/deepin-wine-ubuntu](https://github.com/wszqkzqk/deepin-wine-ubuntu){:target="_blank"}页面下载zip包（或用git方式克隆），解压到本地文件夹，在文件夹中打开终端，输入`sudo sh ./install.sh`一键安装。


---

### 第2步，安装相关应用容器：

第2步，安装相关应用容器：在[http://mirrors.aliyun.com/deepin/pool/non-free/d/](http://mirrors.aliyun.com/deepin/pool/non-free/d/){:target="_blank"}中下载想要的容器，点击deb安装即可。以下为推荐容器，任选其一即可:

```
TIM：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.office/
QQ：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im/
QQ轻聊版：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im.light/

微信：https://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.wechat/
```




下载完成后进入文件下载目录，在终端中执行安装命令：`sudo dpkg -i ****.deb`



---

### 其他deepin发布的最新版容器安装包



> https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu


