---
layout: post
title:  "新松机器人调试与接口"
date:   2019-10-18 00:00:00
categories: 工具
tags: 工具
excerpt: 信科三楼实验室展示机器人调试与接口
mathjax: true
---
* content
{:toc}
---


> [博客地址](https://dufaxing.com){:target="_blank"}


### 机器人内置路由

新松机器人里面使用了小米路由器，其路由密码为：`siasun01`<br/>

- WiFi-name:SIASUN_ERO1_5G
- password:`siasun01`


---

### 配置网络

连接上机器人WiFi后，配置本机网络。

- 查看本机网卡<br/>
```
dufaxing@ubuntu:~$ ifconfig 
ens33     Link encap:Ethernet  HWaddr 00:0c:29:c2:d1:be  
          inet addr:192.168.1.122  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::d609:7497:6022:b8d8/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:442 errors:0 dropped:0 overruns:0 frame:0
          TX packets:258 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:538485 (538.4 KB)  TX bytes:22355 (22.3 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:212 errors:0 dropped:0 overruns:0 frame:0
          TX packets:212 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:16058 (16.0 KB)  TX bytes:16058 (16.0 KB)

```


- `$ vim net.sh `本机使用的是ens33,若不同需要将net.sh中ens33更改为对应的网卡。<br/>

```
#!/bin/sh
export LCM_DEFAULT_URL="udpm://224.0.0.1:7667?ttl=1"
ifconfig ens33 multicast
route add -net 224.0.0.0 netmask 240.0.0.0 dev ens33

```


---

### 标题3:



---
