---
layout: post
title:  "5G-NR架构"
date:   2021-07-18 00:00:00
categories: 5G协议栈
tags: 5G协议栈
excerpt: 5GNR架构
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)



> [博客地址](https://dufaxing.com){:target="_blank"}


# 5G系统网络架构

&emsp;&emsp;5G支持了两种部署方式：一种是分布式部署，这种方式与LTE系统类似，网络由基站组成，基站支持全栈协议的功能；另一种为集中式部署，基站进一步分为集中单元(centralized unit，CU)和分布单元(distributed unit,DU)两个节点，CU和DU分别支持不同的协议栈和功能。5G小基站项目是基于这种集中式的部署。



[![W370cn.jpg](https://z3.ax1x.com/2021/07/18/W370cn.jpg)](https://imgtu.com/i/W370cn)

&emsp;&emsp;5G系统架构任然分为两部分，包括5G核心网（5GC）和5G接入网（NR-RAN）。

&emsp;&emsp;5GC包括AMF、UPF、SMF三种主要逻辑节点。和LTE系统的核心网相比，5G核心网的控制平面和用户平面进一步分离。
&emsp;&emsp;5G核心网对用户平面的控制和转发功能进行了重构，重构后的控制平面分为AMF和SMF两个逻辑节点。AMF主要负责移动性管理，SMF负责会话管理功能。用户平面由UPF构成。



---

# 服务化架构（SBA：Service-based Architecture）

&emsp;&emsp;与4G EPC相比，5G中MME分成AMF、SMF和AUSF三个网元，SGW和PGW分成SMF和UPF。并且新增NSSF、NEF、NRF等网元。

&emsp;&emsp;下图描述了SBA架构下的网元关系图。

[![W89rR0.png](https://z3.ax1x.com/2021/07/18/W89rR0.png)](https://imgtu.com/i/W89rR0)

下图列出了参考点架构下的网元结构图。

[![W89csU.png](https://z3.ax1x.com/2021/07/18/W89csU.png)](https://imgtu.com/i/W89csU)

|  网元   |   描述 | 
|:----:|:----|
|UE|     终端设备，如手机、CPE等      |
|gNB|     5G基站     |	
|AMF|接入和移动管理网元。终结RAN控制面N2接口；终结N1（NAS）接口，实现NAS的加密和完整性保护；UE注册、连接、可达性和移动管理；合法监听；UE与SMF之间SM消息透传和透传路由代理；接入验证、授权管理；UE与SMSF之间SMS消息透传；安全锚点功能；监管服务中的位置服务管理、UE与LMF和RAN与LMF之间的位置服务消息透传；与EPS交互时分配承载ID；UE移动时间通知；CIoT 5GS控制平面优化；根据UE行为和网络配置给UE提供额外参数；策略控制（N15接口）|
|SMF|	会话管理网元。会话建立、修改和释放管理；分配和管理UE IP；DHCPv4\v6客户端和服务端（分别相对真正的DHCP服务端和UE）；回复UE的ARP和IPv6 NS消息；选择UPF、配置UPF数据转发；策略控制；合法监听；计费接口和数据收集；控制UPF计费；终结NAS的SM消息；下行数据通知；发起AN特殊SM信息初始化；决定会话SSC模式；CIoT 5GS控制平面优化；头部压缩；I-SMF；根据UE行为和网络配置给UE提供额外参数；漫游|
|UPF|	数据平面网元。RAT移动锚点；基于SMF请求的UE IP分配；PDU会话与外部网络连接点；数据包路由、转发；DPI；执行策略控制用户面部分；合法监听；用量上报；用户面QoS控制；上行数据检查（SDF与QoS Flow是否匹配）；上下行数据标记（DSCP）；下行数据缓存和通知；发送End Marker；回复UE的ARP和IPv6 NS消息；GTP-U下行数据冗余复制、上行数据消除冗余处理；桥接TSN网络时负责数据翻译|
|PCF|	策略控制网元。统一的策略控制框架；提供策略规则；接入UDR获取用户数据|
|NSSF|	网络切片网元。选择服务UE的网络切片；选择允许的或者配置的NSSAI及与S-NSSAI的映射；根据配置或者询问NRF决定服务UE的AMF组或候补AMF列表。|
|AUSF|	认证服务网元。3GPP和不可信非3GPP接入认证控制|
|UDM|	统一数据管理。生成3GPP AKA认证凭证；用户ID处理（SUPI）；去隐藏受保护的用户ID（SUCI）；基于用户数据实现接入授权；管理UE服务的NF（记录服务UE的AMF、SMF等）；会话、服务连续性；MT-SMS传送；合法监听；签约数据管理；SMS管理；5GLAN组管理；根据UE行为和网络配置给UE提供额外参数|
|UDR|	统一数据仓库。为UDM保存或提取用户数据；为PCF保存或提取策略数据；保存或提取结构化数据；应用数据（如AF请求数据等）|
|CHF|	计费网元。支持UE入网认证、配额授信等，此处不详细描述。|
|NEF|	网络开放网元。能力和事件开放；将AF数据安全地提供给3GPP网络；翻译内外部网络信息；PFD功能；5GLAN组管理|
|NRF|	网络仓库网元。服务发现功能；维护NF配置和服务；|
|AF|	应用网元。|

---

# 协议栈

## 控制面协议栈

下图表示UE、gNB、AMF、SMF、UPF之间的控制面协议栈。
[![W8MKeA.png](https://z3.ax1x.com/2021/07/18/W8MKeA.png)](https://imgtu.com/i/W8MKeA)

## 数据面协议栈

核心网数据面和4G一致，仍是使用GTP-U协议，并通过GTP扩展头的方式携带5G数据面需要的QFI(QoS流ID)参数。
[![W8Maes.png](https://z3.ax1x.com/2021/07/18/W8Maes.png)](https://imgtu.com/i/W8Maes)

---
