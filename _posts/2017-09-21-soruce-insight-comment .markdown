---
layout: post
title:  "Source Insight 配置注释宏"
date:   2017-09-21 15:14:54
categories: 工具
tags: 工具
excerpt: Source Insight4.0环境下，配置注释宏，实现"Ctrl + /"快速注释代码。
mathjax: true
---
* content
{:toc}
---

>Source Insight4.0环境下，配置注释宏，实现"Ctrl + /"快速注释代码。

### step1:
- Project->Open Project <br/>
- 选择Base,然后OK

---

### step2:
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


![](http://wx4.sinaimg.cn/mw690/e4439297gy1fjri9v1p3kj206a07eq2w.jpg)

---

### step3:
**Options->KeyAssignments中你就可以看到这个宏了，宏的名字是MultiLineComments**
![](http://wx2.sinaimg.cn/mw690/e4439297gy1fjri9wkhjzj20fy0epq47.jpg)


### step4:
关闭Base 工程，打开自己的工程。可以测试“Ctrl + /”注释代码，再次“Ctrl + /”取消注释。

---
