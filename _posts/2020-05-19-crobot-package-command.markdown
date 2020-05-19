---
layout: post
title:  "crobot功能包与指令"
date:   2020-05-19 00:00:00
categories: ROS
tags: ROS
excerpt: 由16级石长兴，15级蔡可杰 等师兄搭建的ROS机器人，功能包全部是自主开发与配适，是入门学习ROS机器人不错的硬件平台，基于可传承性，现对crobot代码功能包进行说明与命令备注
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)



> [博客地址](https://dufaxing.com){:target="_blank"}


# 连接crobot

crobot的第二层亚克力板上放置了思科路由器：账号：`cisco`密码：`cisco`

上位机连接crobot路由发射的WiFi<br>
- WiFi:`crobot`<br>
   password:`20180604ymd`<br>

通过ssh远程登录crobot：<br>
- `dufaxing@ubuntu:~$ ssh stone@192.168.1.100`<br>
  `stone@192.168.1.100’s password: 8662182`<br>

以下command以`stone@crobot`开头都指的是通过ssh在crobot远程执行，其他则是在本机执行。

---

# crobot功能包

- crobot_application：为了WIA-PA组采集机器人数据写的脚本，无其他用途
- crobot_bringup：crobot启动相关功能包。
- crobot_description：crobot模型描述相关功能包。
- crobot_mapping：crobot建图相关功能包。
- crobot_navigation:crobot导航相关功能包。
- crobot_teleoperation：crobot键盘控制相关功能包。
- crobot_vision：crobot二维码视觉引导相关功能包。

crobot_package源码:<br>
链接：https://pan.baidu.com/s/1DijsZm5wNugWkor8wuYdmA <br>
提取码：455p

---
# crobot仿真模型

```
//启动机器人模型
dufaxing@ubuntu:~$ roslaunch crobot_bringup fake_crobot.launch 
//在rviz中显示机器人模型
dufaxing@ubuntu:~$ roslaunch crobot_bringup crobot_rviz.launch 
//启动键盘控制
dufaxing@ubuntu:~$ roslaunch crobot_teleoperation keyboard_teleoperation.launch 
```
以上三个命令可以启动crobot仿真模型，并通过键盘控制模型移动。需要注意的是读取键盘需要权限`sudo chmod 777 /dev/input/event1
`,若不清楚键盘对应的event,运行`./src/crobot/crobot_teleoperation/scripts/按键检测/`目录下的`python event_test.py`来检测键盘对应的事件。<br>
启动效果如下图所示<br>
![YhatR1.png](https://s1.ax1x.com/2020/05/18/YhatR1.png)

---

# gmapping建立地图


```
//启动机器人
stone@crobot:~$ roslaunch crobot_bringup crobot.launch 
//启动雷达
stone@crobot:~$ roslaunch crobot_bringup lidar.launch 
//启动gmapping建图
stone@crobot:~$ roslaunch crobot_mapping gmapping.launch 
//本地启动rviz查看建图过程
dufaxing@ubuntu:~$ roslaunch crobot_mapping gmapping_rviz.launch 
//启动键盘，控制机器人移动，方便建图
dufaxing@ubuntu:~$ roslaunch crobot_teleoperation keyboard_teleoperation.launch 
//新建的地图保存在当前目录
dufaxing@ubuntu:~$ rosrun map_server map_saver -f map
```

**注意建图过程中，应使键盘控制crobot速度较慢，同时不允许crobot论坛打滑、空转。**<br>
建图效果如下所示：<br>
![Y43R8f.png](https://s1.ax1x.com/2020/05/19/Y43R8f.png)

---

# 导航

将上一步建图保存的地图信息`map.pgm`,`map.yaml`，放入crobot地图目录下`/home/stone/maps`。

```
//启动机器人
stone@crobot:~$ roslaunch crobot_bringup crobot.launch 
//启动雷达
stone@crobot:~$ roslaunch crobot_bringup lidar.launch 
//启动导航
stone@crobot:~$ roslaunch crobot_navigation navigation.launch
//本地启动rviz查看导航过程
dufaxing@ubuntu:~$ roslaunch crobot_navigation  navigation_rviz.launch 

```

![Y4rF76.png](https://s1.ax1x.com/2020/05/19/Y4rF76.png)


在`2D Nav Goal`发布导航的目标点，机器人将自主导航至目标。

![Y4rs4U.png](https://s1.ax1x.com/2020/05/19/Y4rs4U.png)


---