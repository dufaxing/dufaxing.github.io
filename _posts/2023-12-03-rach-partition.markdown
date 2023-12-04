---
layout: post
title:  "随机接入与preamble分隔"
date:   2023-12-03 00:00:00
categories: 5G协议栈
tags: NR
excerpt: 3GPP R17特性
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)



> [博客地址](https://dufaxing.com){:target="_blank"}


# SSB与PRACH

NR中，下行广行播信息SSB/RMSI,初始接入也可以支持波束Beam管理机制;

SSB在时域周期内有多次发送机会，可以分别对应不同波束;

因此NR中，只有当SSB的波束扫描信号“ 覆盖"到UE时，UE才 有机会发送PRACH随机接入。

即: PRACH的发送时刻(RO)需要和SSB发送的时刻(索引) 建立映射关系。同时基站根据UE上行PRACH的资源位置，决定下行RAR发送的波束。



RACH参数配置中ssb perRACH-OccasionAndCB-PreamblesPerSSB用于配置:
    - 每个RACH时刻对应的SSB个数N,从1/8-16
    - 当ssb-perRACH-Occasion>=1, N>=1时，即多个SSB对应1个RACH Occasion,从n*64/N开始的连续CB PreamblesPerSSB个CB preambles对应于SSB n, 0<=n<=N-1 。


- ssb-perRACH-Occasion = 1/8，n=60
    - 一个SSB映射到8个RO
    - 每个RO.上的CB Preamble分别为0-59
    - 每个RO.上的CB Preamble分别为60-63

[![pi6C0JA.png](https://z1.ax1x.com/2023/12/04/pi6C0JA.png)](https://imgse.com/i/pi6C0JA)

- ssb-perRACH-Occasion = 1, n=56
    - 一个SSB映射到1个RO
    - 每个RO.上的CB Preamble分别为0-55
    - 每个RO.上的CF Preamble分别为56-63

[![pi6CBRI.png](https://z1.ax1x.com/2023/12/04/pi6CBRI.png)](https://imgse.com/i/pi6CBRI)

- ssb-perRACH-Occasion = 4, n=12
    - 4个SSB映射到一个RO，
    - 对应的CB Preamble分别为0-11，16-27,32-43, 48-59
    - 对应的CF Preamble分别为12-15，28-31,44-47, 60-63


[![pi6CDzt.png](https://z1.ax1x.com/2023/12/04/pi6CDzt.png)](https://imgse.com/i/pi6CDzt)



---

# 标题2:




---

# 标题3:



---
