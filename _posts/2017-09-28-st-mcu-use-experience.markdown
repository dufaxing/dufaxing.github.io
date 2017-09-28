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
void   PORT_Init(void) {
	GPIOD->ODR=0x00;//0000 0000
	GPIOD->DDR=0x20;//0010 0000
	GPIOD->DDR|=0x04;//0010 0100
	GPIOD->CR1=0xFF;//1111 1111
	GPIOD->CR2=0x20;//0000 0000	
}

```
---

### 读取内存地址的值:

```
unsigned char read_port_PB4() {
	unsigned char p;
	p = *(unsigned char *)(0x5006);//PB_IDR
	unsigned GPIO_PB4 = (p & 0x10)>>4;
	return GPIO_PB4;
}

```
在定义p时，定义为unsigned char 型，我一开始定义为`unsigned char p;p = *(unsigned char *)(0x5006);//PB_IDR`会读取到下一个内存地址的值。

读取内存地址的值，可以用下面的方法来读取
```
int *p=(int *)0x123456;//addr
int result=*p;
```

---

### END:


---
