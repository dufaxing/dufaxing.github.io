---
layout: post
title:  "ROS远程连接至机器人"
date:   2019-2-22 00:00:00
categories: ROS
tags: ROS
excerpt: NULL
mathjax: true
---
* content
{:toc}
---



### 配置bashrc文件


- 设置工作区的环境变量</br>
```
source /home/dufaxing/Project/my_robot/devel/setup.bash
```

- 设置主机名
 - 获取IP地址
 
 ```
 dufaxing@ubuntu:~$ ifconfig
ens33     Link encap:Ethernet  HWaddr 00:0c:29:c2:d1:be  
          inet addr:172.18.17.152  Bcast:172.18.19.255  Mask:255.255.252.0
          inet6 addr: fe80::d609:7497:6022:b8d8/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:880 errors:0 dropped:0 overruns:0 frame:0
          TX packets:162 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:56852 (56.8 KB)  TX bytes:14845 (14.8 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:678 errors:0 dropped:0 overruns:0 frame:0
          TX packets:678 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:76798 (76.7 KB)  TX bytes:76798 (76.7 KB)
```
 



---

### 标题2:




---

### 标题3:



---
