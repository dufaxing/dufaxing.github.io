---
layout: post
title:  "101机械臂IP地址配置、路由密码、PC密码"
date:   2019-11-20 00:00:00
categories: 工具
tags: 工具
excerpt: 为了使101机械臂实验室IP Adress不冲突，对各个设备IP地址进行了分配
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)



> [博客地址](https://dufaxing.com){:target="_blank"}

# IP地址

- `PLC IP:192.168.1.10`  

- `机械臂 IP:192.168.1.20` 

- `TP-link交换机 IP:192.168.1.1` 

- `someone路由器 IP:192.168.1.2` 

- `AGV IP:192.168.1.104` 

- `PC_wifi IP:192.168.1.105` 


# 交换机与路由密码

- TP-link交换机:
    - `管理员账号：tplink`
    - `管理员密码：tplink`

- AGV上华为路由：
    - `wifi:视觉导航机器人 `
    - `Passwd：BlueWhale`
    - `管理员密码：BlueWhale`

- 101实验室分流水星路由器：
    - `wifi:someone `
    - `Passwd：20171225ymd`
    - `管理员密码：8662182`

- crobot移动机器人内置思科路由：
    - `wifi:crobot `
    - `Passwd：20180604ymd`
    - `管理员账号：cisco`
    - `管理员密码：cisco`

- 新松语音播报移动机器人内置小米路由：
    - `wifi:SIASUN_ERO1_5G `
    - `Passwd：siasun01`
    - `管理员密码：siasun01`

---

# PC机密码

- 视觉PC机：
    - `开机密码：554262`

- 组态软件PC机：
    - `开机密码：0000`

- PLC控制台：
    - `管理员密码：1122`


---

# 海康威视巡逻摄像头

- 首先连上AGV上内置路由器，用ie浏览器登录摄像头网页管理界面，安装插件后才可以用视觉导航客户端浏览图像
- 登录地址：`http://192.168.0.198`
- 用户名: `admin`
- 密码：`SctecCam01`