---
layout: post
title:  "ROS Network配置"
date:   2019-2-22 00:00:00
categories: ROS
tags: ROS
excerpt: ros网络配置远程连接至远程机器人
mathjax: true
---
* content
{:toc}
---

> 我们知道，在运行所有的ROS节点之前，一定要先使用指令roscore，实际上roscore指令是运行了一个ros master，ros master相当于一个服务器，所有节点都可以通过ros master发布消息和订阅消息，还包括发布和订阅服务，以及参数服务，只有连接在相同ros master下的节点才能互相通信。 
怎样才能知道节点是运行在哪个master下的呢？实际上ROS中默认一个机器上只运行一个ros master，其运行的端口默认为http://localhost:11311,这个端口默认值存于ROS环境变量ROS_MASTER_URI中，可以通过下面指令查看检验，或者在运行roscore的命令行输出中也可以看到</br>
`echo $ROS_MASTER_URI`</br>


```
dufaxing@ubuntu:~$ echo $ROS_MASTER_URI
http://localhost:11311
```

即使是不同机器上的程序节点，只要通过网络连接到ros master的运行端口，就能实现消息的发布和订阅，并且发布的消息就能被所有连接到ros master端口的机器和节点所订阅，同样也可以订阅连接到这个端口的其他节点所发布的消息。有了这样的通讯方式，我们完全不用往我们的程序节点中添加任何与通讯相关的代码，就能订阅到其他机器发布的话题，与获取本机上其他节点发布的话题没有任何区别。 


### 配置hosts文件



- 获取本机名

```
dufaxing@ubuntu:~$ hostname
ubuntu
```

- 获取本机IP<br/>

`dufaxing@ubuntu:~$ ifconfig`<br/>

```
ens33     Link encap:Ethernet  HWaddr 00:0c:29:d7:36:58  
          inet addr:192.168.1.105  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::c61e:539f:d42f:7137/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:617351 errors:0 dropped:0 overruns:0 frame:0
          TX packets:359629 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:589000305 (589.0 MB)  TX bytes:58190976 (58.1 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:1261 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1261 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:252666 (252.6 KB)  TX bytes:252666 (252.6 KB)

```



- 在本地加入crobot的IP地址<br/>



`dufaxing@ubuntu:~$ sudo vim /etc/hosts`<br/>

```
127.0.0.1       localhost
127.0.1.1       ubuntu

192.168.1.100   crobot

```


- 在从robot中加入上位机的IP地址

```
127.0.0.1       localhost
127.0.1.1       crobot

192.168.1.105   ubuntu
```


### 配置bashrc文件



- 设置工作区的环境变量</br>

```
source /home/dufaxing/Project/my_robot/devel/setup.bash
```



 

- bashrc中添加环境变量：<br/>


`dufaxing@ubuntu:~$ vim ~/.bashrc `

```
export ROS_IP=ubuntu
export ROS_MASTER_URI=http://crobot:11311
```

- 在远程端(crobot)中添加环境变量：<br/>
```
export ROS_HOSTNAME=crobot
export ROS_MASTER_URI=http://crobot:11311
```


- 重新加载bashrc配置文件

`dufaxing@ubuntu:~$ source ~/.bashrc `


---

### ssh远程登陆

`ssh user_name@192.168.1.xxx  -X`通过-X登录和平时ssh连接没有什么区别，除了会转发X11的数据，你可以在终端里面用命令运行你想要运行的gui程序。



---
