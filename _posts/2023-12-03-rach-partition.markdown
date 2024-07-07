---
layout: post
title:  "随机接入与preamble选择"
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

# BWP

部分带宽（BWP）是在给定载波和给定 Numerology 条件下的一组连续的PRB。由于 NR 支持小至 5 MHz、大至 400 MHz 的工作带宽，如果要求所有UE 均支持最大的 400 MHz 带宽，无疑会对 UE 的性能提出较高要求，也不利于降低 UE 的成本。同时，由于一个 UE 不可能同时占满整个 400 MHz 带宽，且高带宽意味着高采样率，而高采样率意味着更高功耗，如果 UE 全部按照支持 400 MHz 的带宽进行设计，无疑是对性能的极大浪费。因此，NR 引入了带宽自适应（Bandwidth Adaptation）技术，针对性地解决上述问题。带宽自适应意味着，UE 在低业务周期可以使用适度的带宽监测控制信道，而只在必要时才启用大的接收带宽以应对高业务负荷。
在 LTE 中，UE 的带宽与系统带宽保持一致，在解码 MIB 信息配置带宽后便保持不变。而在 NR 中，不同的 UE 可以配置不同的 BWP，也就是说，UE 的带宽可以动态变化。如图 3-26 所示，在 T0 时段，UE 业务负荷较大且对时延要求不敏感，系统为 UE 配置大带宽 BWP1（BW 为 40 MHz，SCS 为15 kHz）；在 T1 时段，由于业务负荷趋降，UE 由 BWP1 切换至小带宽 BWP2（BW 为 10 MHz，SCS 为 15 kHz），在满足基本通信需求的前提下，可达到减低功耗的目的；在 T2 时段，UE 可能突发时延敏感业务，或者发现 BWP1 所在频段内资源紧缺，于是切换到新的 BWP3（BW 为 20 MHz，SCS 为 60 kHz）上；同理，在 T3 和 T4 等其他不同时段，UE 均根据实时业务需求，在不同 BWP之间切换。



[![pkR9Hwn.png](https://s21.ax1x.com/2024/07/04/pkR9Hwn.png)](https://imgse.com/i/pkR9Hwn)

## BWP的分类
BWP 具体可以分为 Initial BWP 和 Dedicated BWP 两类，其中，Initial BWP是 UE 在初始接入阶段使用的 BWP，主要用于发起随机接入等。Dedicated BWP是 UE 在 RRC 连接态时配置的 BWP，主要用于数据业务传输。根据 R15，一个 UE 可以通过 RRC 信令分别在上、下行链路各自独立配置最多 4 个 DedicatedBWP，如果 UE 配置了 SUL，则在 SUL 链路上可以额外配置最多 4 个 Dedicated BWP。需要特别指出的是，对于 NR TDD 系统，DL BWP 和 UL BWP 是成对的，其中心频点保持一致，但带宽和子载波间隔的配置可以不同。<br/>
UE 在 RRC 连接态时，某一时刻有且只能激活一个 Dedicated BWP，称为Active BWP。当其 BWPinactivitytimer 超时，UE 所工作的 Dedicated BWP，称为 Default BWP。图 3-30 示出了 BWP 的分类及切换流程的示意。
[![pkRCSOJ.png](https://s21.ax1x.com/2024/07/04/pkRCSOJ.png)](https://imgse.com/i/pkRCSOJ)

---

# SSB与PRACH

NR中，下行广行播信息SSB/RMSI,初始接入也可以支持波束Beam管理机制;

SSB在时域周期内有多次发送机会，可以分别对应不同波束;

因此NR中，只有当SSB的波束扫描信号“ 覆盖"到UE时，UE才 有机会发送PRACH随机接入。

即: PRACH的发送时刻(RO)需要和SSB发送的时刻(索引) 建立映射关系。同时基站根据UE上行PRACH的资源位置，决定下行RAR发送的波束。

[![pkWUGAH.png](https://s21.ax1x.com/2024/07/07/pkWUGAH.png)](https://imgse.com/i/pkWUGAH)

[![pkWUJNd.png](https://s21.ax1x.com/2024/07/07/pkWUJNd.png)](https://imgse.com/i/pkWUJNd)

# RACH Occasion

> RACH Occasion is an area specified in time and frequency domain that are available for the reception of RACH preamble.

RACH Occasion是一种特定的时域和频域资源，可用于接收RACH前导码。

在LTE中，对于所有可能的preamble码都使用SIB2中规定的RO。

在NR中，同步信号（SSB）与不同的波束相关联，UE选择某个波束并使用该波束发送PRACH。为了让 网络 弄清楚 UE 选择了哪个波束，3GPP 定义了 SSB 和 RO之间的特定映射。通过确定 UE在哪个RO上发送了PRACH，NW 就可以确定 UE 选择了哪个 SSB 波束。

SSB 和 RACH Occasion 之间的映射由以下两个 RRC 参数定义：

- msg1-FDM
- ssb-perRACH-OccasionAndCB-PreamblesPerSSB

msg-FDM 指定在频域中分配了多少个 RO（在时域中的同一位置）。
ssb-perRACH-OccasionAndCB-PreamblesPerSSB 指定可以映射到一个RO的SSB数量，以及可以映射到一个SSB的preamble的数量。

> 38.213-8.1:SS/PBCH block indexes provided by ssb-PositionsInBurst in SIB1 or in ServingCellConfigCommon are mapped to valid PRACH occasions in the following order where the parameters are described in [4, TS 38.211].
>-	First, in increasing order of preamble indexes within a single PRACH occasion
>-	Second, in increasing order of frequency resource indexes for frequency multiplexed PRACH occasions
>-	Third, in increasing order of time resource indexes for time multiplexed PRACH occasions within a PRACH slot
>-	Fourth, in increasing order of indexes for PRACH slots

## 举例
下图38.331 对于的定义
[![pkWUA74.jpg](https://s21.ax1x.com/2024/07/07/pkWUA74.jpg)](https://imgse.com/i/pkWUA74)

### Example 01 

msg1-FDM = one
ssb-perRACH-OccasionAndCB-PreamblesPerSSB = one

[![pkWUu1x.png](https://s21.ax1x.com/2024/07/07/pkWUu1x.png)](https://imgse.com/i/pkWUu1x)

### Example 02 

msg1-FDM = two
ssb-perRACH-OccasionAndCB-PreamblesPerSSB = one

[![pkWUKc6.png](https://s21.ax1x.com/2024/07/07/pkWUKc6.png)](https://imgse.com/i/pkWUKc6)

### Example 03 

msg1-FDM = two
ssb-perRACH-OccasionAndCB-PreamblesPerSSB = eight

[![pkWUlnO.png](https://s21.ax1x.com/2024/07/07/pkWUlnO.png)](https://imgse.com/i/pkWUlnO)

### Example 04 

msg1-FDM = two
ssb-perRACH-OccasionAndCB-PreamblesPerSSB = oneHalf

[![pkWU3He.png](https://s21.ax1x.com/2024/07/07/pkWU3He.png)](https://imgse.com/i/pkWU3He)

---

# Legacy RA

高层通过参数`ssb-perRACH-OccasionAndCB-PreamblesPerSSB`配置`N`个SSB关联一个PRACH occasion(频域)，和每个SSB在每个有效PRACH occasion上基于竞争的preamble数(参数：`CB-preambles-per-SSB`)。其中对于N的配置有如下两种：

> 38.213-i30：
For Type-1 random access procedure, a UE is provided a number N of SS/PBCH block indexes associated with one PRACH occasion and a number R of contention based preambles per SS/PBCH block index per valid PRACH occasion by ssb-perRACH-OccasionAndCB-PreamblesPerSSB. 


[![pk2azz6.png](https://s21.ax1x.com/2024/07/03/pk2azz6.png)](https://imgse.com/i/pk2azz6)

[![pk2dotA.png](https://s21.ax1x.com/2024/07/03/pk2dotA.png)](https://imgse.com/i/pk2dotA)

说明
RACH参数配置中ssb-perRACH-OccasionAndCB-PreamblesPerSSB用于配置:
    - 每个RACH时刻对应的SSB个数N,从1/8-16
    - 当ssb-perRACH-Occasion>=1, N>=1时，即多个SSB对应1个RACH Occasion,从n*N<sup>total</sup><sub>preamble</sub>/N开始的连续CB PreamblesPerSSB个CB preambles对应于SSB n, 0<=n<=N-1 。

## 举例

- ssb-perRACH-Occasion = 1/8，totalNumberOfRA-Preambles=60
    - 一个SSB映射到8个RO
    - 每个RO.上的CB Preamble分别为0-59
    - 每个RO.上的CB Preamble分别为60-63

[![pi6C0JA.png](https://z1.ax1x.com/2023/12/04/pi6C0JA.png)](https://imgse.com/i/pi6C0JA)

- ssb-perRACH-Occasion = 1, totalNumberOfRA-Preambles=56
    - 一个SSB映射到1个RO
    - 每个RO.上的CB Preamble分别为0-55
    - 每个RO.上的CF Preamble分别为56-63

[![pi6CBRI.png](https://z1.ax1x.com/2023/12/04/pi6CBRI.png)](https://imgse.com/i/pi6CBRI)

- ssb-perRACH-Occasion = 4, totalNumberOfRA-Preambles=12
    - 4个SSB映射到一个RO，
    - 对应的CB Preamble分别为0-11，16-27,32-43, 48-59
    - 对应的CF Preamble分别为12-15，28-31,44-47, 60-63


[![pi6CDzt.png](https://z1.ax1x.com/2023/12/04/pi6CDzt.png)](https://imgse.com/i/pi6CDzt)



---

# RACH partition



---
