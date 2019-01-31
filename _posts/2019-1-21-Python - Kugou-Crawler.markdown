---
layout: post
title:  "Python爬取酷狗TOP500的数据"
date:   2019-1-31 00:00:00
categories: Python
tags: 爬虫
excerpt: 利用 Requests 和 BeautifulSoup 第三方库，爬取酷狗榜单中的酷狗TOP500的信息
mathjax: true
---
* content
{:toc}
---



### 爬虫思路分析

爬取网址：[http://www.kugou.com/yy/rank/home/1-8888.html](http://www.kugou.com/yy/rank/home/1-8888.html){:target="_blank"}

网页版的酷狗不能手动翻页进行下一步的浏览，但通过观察第一页的URL:
< 'http://www.kugou.com/yy/rank/home/1-8888.html'  
< 'http://www.kugou.com/yy/rank/home/2-8888.html'  


尝试把数字1改成2，发现恰好返回的是第二页的信息。所以爬取TOP500的歌曲只需要改home后面的数字即可。




---

### 安装第三方库

这里的实验环境是PyCharm。

- 1. 打开File->Default Settings


[![k1aWtg.png](https://s2.ax1x.com/2019/01/31/k1aWtg.png)](https://imgchr.com/i/k1aWtg)


- 2. 按图所示，打开Project Interpreter选择工程，在点击加号，添加‘beautifulsoup4’,'lxml','requests'库。

[![k1a4pj.png](https://s2.ax1x.com/2019/01/31/k1a4pj.png)](https://imgchr.com/i/k1a4pj)


---

### 标题3:



---
