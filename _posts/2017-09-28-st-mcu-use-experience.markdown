---
layout: post
title:  "一款STM8的使用记录"
date:   2017-09-28 00:00:00
categories: 嵌入式
tags: STM
excerpt: 近几天工作中使用到了一款STM8的芯片，工程中没有使用库函数，甚至没有读取引脚电平的函数。
mathjax: true
---
* content
{:toc}
---



### 端口初始化:

![](http://owlypioka.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20170928161904.png)

阅读数据手册看到，某个引脚若要配置上拉输入，DDR寄存器置0，CR1寄存器置1，CR2寄存器置0.

![](http://owlypioka.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20170928163249.png)

某个引脚若要配置推挽输出，DDR寄存器置1，CR1寄存器置1，CR2寄存器置0.

```
void   PORT_Init(void)
{

	GPIOD->ODR=0x00;//0000 0000
	GPIOD->DDR=0x20;//0010 0000
	GPIOD->DDR|=0x04;//0010 0100
	GPIOD->CR1=0xFF;//1111 1111
	GPIOD->CR2=0x20;//0000 0000	
}

```
---

### 标题2:




---

### 标题3:


---
