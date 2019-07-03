---
layout: post
title:  "在Linux中绑定USB设备名"
date:   2019-5-26 00:00:00
categories: linux
tags: 工具
excerpt: 机器人运行在基于Linux的ROS机器人系统中，跟底层通过USB设备交互，但是Linux设备启动顺序不确定，通过USB号访问设备很麻烦
mathjax: true
---
* content
{:toc}
---


> 移动机器人上有两个USB串口设备，分别是激光雷达和机器人底盘，Ubuntu为这两个设备的设备名分配的设备名ttyUSB0和ttyUSB1，但是设备名与设备之间的对应关系并不是固定的，是按设备接入系统的顺序依次分配的。先插入一个设备再插另一个设备可以确定设备与设备名的关系，但每次系统启动都需要插拔设备，十分麻烦。<br/>
可以将串口映射到一个固定的设备名上，不论插入顺序如何都会讲设备映射到新的设备名上，我们只要使用新的设备名对设备进行读写操作就可以了。<br/>



### 查看USB端口信息

`$ lsusb`

![ZtEZdK.png](https://s2.ax1x.com/2019/07/03/ZtEZdK.png)


第一个设备是移动机器人底盘，第二个设备是激光雷达。我们需要的是这两个设备的ID，分别是`1a86:7523`和`10c4:ea60`。

---

### 建立端口映射关系

- 创建base映射关系<br/>


`$ sudo vim /etc/udev/rules.d/base.rules`<br/>

在文件中添加如下内容：<br/>


`KERNEL=="ttyUSB*", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE:="0666", SYMLINK+="base"  `<br/>


- 创建lidar映射关系<br/>


`$ sudo vim /etc/udev/rules.d/lidar.rules`<br/>


在文件中添加如下内容：<br/>

`KERNEL=="ttyUSB*", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", MODE:="0666", SYMLINK+="lidar"`<br/>


---

### 查看映射结果

`$ ll /dev | grep ttyUSB`

![ZtElQA.png](https://s2.ax1x.com/2019/07/03/ZtElQA.png)

---
