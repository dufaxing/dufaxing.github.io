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

网页版的酷狗不能手动翻页进行下一步的浏览，但通过观察第一页的URL:<br/>

> `http://www.kugou.com/yy/rank/home/1-8888.html` <br/>
> `http://www.kugou.com/yy/rank/home/2-8888.html`<br/>


尝试把数字1改成2，发现恰好返回的是第二页的信息。所以爬取TOP500的歌曲只需要改home后面的数字即可。




---

### 安装第三方库

这里的实验环境是PyCharm。

- 1. 打开File->Default Settings


[![k1aWtg.png](https://s2.ax1x.com/2019/01/31/k1aWtg.png)](https://imgchr.com/i/k1aWtg)


- 2. 按图所示，打开Project Interpreter选择工程，在点击加号，添加`beautifulsoup4`,`lxml`,`requests`库。

[![k1a4pj.png](https://s2.ax1x.com/2019/01/31/k1a4pj.png)](https://imgchr.com/i/k1a4pj)


---

### request headers的获取

有时爬虫需要加入请求头来伪装成浏览器，以便于更好的抓取数据。在浏览器中按F12打开开发者工具，刷新后找到`User-Agent`进行复制，如图所示。

[![k1wfQs.png](https://s2.ax1x.com/2019/01/31/k1wfQs.png)](https://imgchr.com/i/k1wfQs)



---

### 完整代码


```
import requests
from bs4 import BeautifulSoup
import time

headers = {
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3294.6 Safari/537.36'}


##获取信息
def get_info(url):
    r = requests.get(url, headers=headers)
    print(r.status_code)
    soup = BeautifulSoup(r.text, "lxml")

    ranks = soup.select("span.pc_temp_num")
    # for rank in ranks:
    #     print(rank.text.strip())

    titles = soup.select("a.pc_temp_songname")
    # for title in titles:
    #     singer = title.text.split("-")[0].strip()
    #     song = title.text.split("-")[1].strip()
    #     print(singer,song)

    times = soup.select("span.pc_temp_time")
    # for time in times:
    #     time = time.text.strip()
    #     print(time)

    for rank, title, time in zip(ranks, titles, times):
        data = {
            'rank': rank.text.strip(),
            'singer': title.text.split("-")[0].strip(),
            'song': title.text.split("-")[1].strip(),
            'time': time.text.strip()
        }
        print(data)


if __name__ == "__main__":
    urls = ["http://www.kugou.com/yy/rank/home/{}-8888.html".format(i) for i in range(1, 24)]  ##构造URL的请求页面
    for url in urls:
        get_info(url)
        time.sleep(1)
```



[![k10CFO.png](https://s2.ax1x.com/2019/01/31/k10CFO.png)](https://imgchr.com/i/k10CFO)