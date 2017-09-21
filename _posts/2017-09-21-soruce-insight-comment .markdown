---
title: "Source Insight 配置注释宏"
layout: post
date: 2017-09-21 10:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Source Insight
- tool
category: blog
author: dufaxing
description: Source Insight4.0环境下，配置注释宏，实现‘Ctrl + /’快速注释代码
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'

---

# 摘要

[TOC]

>Source Insight4.0环境下，配置注释宏，实现"Ctrl + /"快速注释代码。

### 1:
<br>打开source insight文件夹，utils.em文件
![](http://chuantu.biz/t6/57/1505961964x2890171582.png)

---

### 2:
<br>在utils.em中添加代码

```
macro MultiLineComment()
{
    hwnd = GetCurrentWnd()
    selection = GetWndSel(hwnd)
    LnFirst =GetWndSelLnFirst(hwnd)      //取首行行号
    LnLast =GetWndSelLnLast(hwnd)      //取末行行号
    hbuf = GetCurrentBuf()
    if(GetBufLine(hbuf, 0) =="//magic-number:tph85666031"){
        stop	
    }
    Ln = Lnfirst
    buf = GetBufLine(hbuf, Ln)
    len = strlen(buf)
    while(Ln <= Lnlast) {
        buf = GetBufLine(hbuf, Ln)  //取Ln对应的行
        if(buf ==""){                   //跳过空行
            Ln = Ln + 1	
            continue
        }
        if(StrMid(buf, 0, 1) == "/"){       //需要取消注释,防止只有单字符的行
            if(StrMid(buf, 1, 2) == "/"){
                PutBufLine(hbuf, Ln, StrMid(buf, 2, Strlen(buf)))	
            }	
        }
        if(StrMid(buf,0,1) !="/"){          //需要添加注释
            PutBufLine(hbuf, Ln, Cat("//", buf))	
        }
        Ln = Ln + 1
    }
    SetWndSel(hwnd, selection)	
}
```


![](http://chuantu.biz/t6/57/1505963258x2890191691.png)

---

### 3:
**Options->KeyAssignments中你就可以看到这个宏了，宏的名字是MultiLineComments**
![](http://chuantu.biz/t6/57/1505962833x2890174154.png)

---
