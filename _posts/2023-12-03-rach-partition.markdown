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

# 小区搜索与RACH流程

[![pkf6DjH.png](https://s21.ax1x.com/2024/07/09/pkf6DjH.png)](https://imgse.com/i/pkf6DjH)




# BWP

&emsp;&emsp;部分带宽（BWP）是在给定载波和给定 Numerology 条件下的一组连续的PRB。由于 NR 支持小至 5 MHz、大至 400 MHz 的工作带宽，如果要求所有UE 均支持最大的 400 MHz 带宽，无疑会对 UE 的性能提出较高要求，也不利于降低 UE 的成本。同时，由于一个 UE 不可能同时占满整个 400 MHz 带宽，且高带宽意味着高采样率，而高采样率意味着更高功耗，如果 UE 全部按照支持 400 MHz 的带宽进行设计，无疑是对性能的极大浪费。因此，NR 引入了带宽自适应（Bandwidth Adaptation）技术，针对性地解决上述问题。带宽自适应意味着，UE 在低业务周期可以使用适度的带宽监测控制信道，而只在必要时才启用大的接收带宽以应对高业务负荷。<br/>
&emsp;&emsp;在 LTE 中，UE 的带宽与系统带宽保持一致，在解码 MIB 信息配置带宽后便保持不变。而在 NR 中，不同的 UE 可以配置不同的 BWP，也就是说，UE 的带宽可以动态变化。如图 3-26 所示，在 T0 时段，UE 业务负荷较大且对时延要求不敏感，系统为 UE 配置大带宽 BWP1（BW 为 40 MHz，SCS 为15 kHz）；在 T1 时段，由于业务负荷趋降，UE 由 BWP1 切换至小带宽 BWP2（BW 为 10 MHz，SCS 为 15 kHz），在满足基本通信需求的前提下，可达到减低功耗的目的；在 T2 时段，UE 可能突发时延敏感业务，或者发现 BWP1 所在频段内资源紧缺，于是切换到新的 BWP3（BW 为 20 MHz，SCS 为 60 kHz）上；同理，在 T3 和 T4 等其他不同时段，UE 均根据实时业务需求，在不同 BWP之间切换。



[![pkR9Hwn.png](https://s21.ax1x.com/2024/07/04/pkR9Hwn.png)](https://imgse.com/i/pkR9Hwn)

## BWP的分类
&emsp;&emsp; BWP 具体可以分为 Initial BWP 和 Dedicated BWP 两类，其中，Initial BWP是 UE 在初始接入阶段使用的 BWP，主要用于发起随机接入等。Dedicated BWP是 UE 在 RRC 连接态时配置的 BWP，主要用于数据业务传输。根据 R15，一个 UE 可以通过 RRC 信令分别在上、下行链路各自独立配置最多 4 个 DedicatedBWP，如果 UE 配置了 SUL，则在 SUL 链路上可以额外配置最多 4 个 Dedicated BWP。需要特别指出的是，对于 NR TDD 系统，DL BWP 和 UL BWP 是成对的，其中心频点保持一致，但带宽和子载波间隔的配置可以不同。<br/>
&emsp;&emsp;UE 在 RRC 连接态时，某一时刻有且只能激活一个 Dedicated BWP，称为Active BWP。当其 BWPinactivitytimer 超时，UE 所工作的 Dedicated BWP，称为 Default BWP。图 3-30 示出了 BWP 的分类及切换流程的示意。
[![pkRCSOJ.png](https://s21.ax1x.com/2024/07/04/pkRCSOJ.png)](https://imgse.com/i/pkRCSOJ)

---

# 波束与接入过程

&emsp;&emsp;波束互易性是指一个设备可以根据接收波束推导准确发射波束；或者由发射波束来推导确定接收波束的特性。<br/>

&emsp;&emsp;随机接入过程需要确定的波束包括：UE发送MSG1和MSG3的发送波束，UE接收MSG2和MSG4的接受波束，基站接收MSG1和MSG3的接收波束，基站发送MSG2和MSG4的发送波束。需要注意的是，随机接入过程中PRACH的波束既可以基于SSB也可以基于CSI-RS测量。本文只讨论基于SSB测量的随机接入UE发送MSG1的流程。<br/>

&emsp;&emsp;UE在发送Msg1之前已经实现了下行的同步，检测到了一个满足检测门限的SSB，并通过该 SSB 关联的 SIB1 获得了PRACH 发送的發源配置信息。UE 在检测SSB 的过程中已经确定了接收信号的波束。从数据传输的角度，如果基站用该SSB 的发送波束发送下行数据，
UE用对应的接收波束接收数据，可实现下行数据的传输。<br/>
&emsp;&emsp;如果UE 具有波束互易性，UE 就通过接收波束确定一个发送波束用于发送 Preamble。如果UE 不具备波束互易性，只能采用逐个尝试的方式发送 Preamble，即上行波束扫描。<br/>
&emsp;&emsp;基站通过接收波束赋形实现对UB 发送的 Preamble 的接收，如果基站的波束互易性不成立，则基站需要对多个可能的接收波束进行尝试，即采用接收波束扫描的方式接收信号。<br/>
&emsp;&emsp;如果基站波束互易性成立，基站就可以由发送波束确定对应的接收波束。UE 在发送Preamble 之前，已经选择了一个SSB，该SSB 的发送波束是确定基站接收波束的最佳候选，但此时 UE 并未通知基站所选择的SSB。NR给出的解决方案是建立 RO 子集以及 Preamble和SSB的关联关系，UE 检测到一个 SSB之后从该 SSB 关联的 RO子集以及 Preamble 中速择RO和 Preamble。UE 选择的 RO 和 Preamble 隐含地指示了UE 选择的SSB，基站在特定的PRACH 资源上检测特定的Preamble 时就可以用其关联的 SSB 的发送波束确定接收波束。<br/>
&emsp;&emsp;基站按照这个方式检测的另一个前提条件是 UE 的波束互易性也成立。如果 UE 的波束互易性不成立，UB 选择的发送波束和接收SSB 的接收波束不一定是相同方向，从而导致下行传输的路径和上行传输的路径有差别，，那么基站的接收波束也会有变化，基站仍然要进行波束扫描。<br/>
&emsp;&emsp;具体来说，SSB 与 RO 子集关联关系由基站通过系统广播消息通知 UE（包括选择 SSB的RSRP 阈值），其中，一个 RO 子集由一个或者多个 RO组成。UE 检测SSB，并判断 SSB的实际 RSRP 测量值是否大于从系统广播消息获取的 RSRP 阈值，如果判断成立，则使用该SSB 关联的RO子集作 候选的 RO集合。如果有多个 SSB 的RSRP 测量值大于阈值，则UE 从中随机选择一个SSB，使用该 SSB 关联的 RO 子集作为候选的RO集合。基站根据检测到 Preamble 的 RO与SSB（下行发送波束）的关联关系，确定 UE 选择的SSB（下行发送波束）。<br/>

---

# SSB与PRACH

&emsp;&emsp;NR中，下行广行播信息SSB/RMSI,初始接入也可以支持波束Beam管理机制;

&emsp;&emsp;SSB在时域周期内有多次发送机会，可以分别对应不同波束;

&emsp;&emsp;因此NR中，只有当SSB的波束扫描信号“ 覆盖"到UE时，UE才 有机会发送PRACH随机接入。

&emsp;&emsp;即: PRACH的发送时刻(RO)需要和SSB发送的时刻(索引) 建立映射关系。同时基站根据UE上行PRACH的资源位置，决定下行RAR发送的波束。

[![pkWUGAH.png](https://s21.ax1x.com/2024/07/07/pkWUGAH.png)](https://imgse.com/i/pkWUGAH)

[![pkWUJNd.png](https://s21.ax1x.com/2024/07/07/pkWUJNd.png)](https://imgse.com/i/pkWUJNd)

# RACH Occasion

> RACH Occasion is an area specified in time and frequency domain that are available for the reception of RACH preamble.

&emsp;&emsp;RACH Occasion是一种特定的时域和频域资源，可用于接收RACH前导码。

&emsp;&emsp;在LTE中，对于所有可能的preamble码都使用SIB2中规定的RO。

&emsp;&emsp;在NR中，同步信号（SSB）与不同的波束相关联，UE选择某个波束并使用该波束发送PRACH。为了让 网络 弄清楚 UE 选择了哪个波束，3GPP 定义了 SSB 和 RO之间的特定映射。通过确定 UE在哪个RO上发送了PRACH，NW 就可以确定 UE 选择了哪个 SSB 波束。

SSB 和 RACH Occasion 之间的映射由以下两个 RRC 参数定义：

- msg1-FDM
- ssb-perRACH-OccasionAndCB-PreamblesPerSSB

&emsp;&emsp;msg-FDM 指定在频域中分配了多少个 RO（在时域中的同一位置）。
&emsp;&emsp;ssb-perRACH-OccasionAndCB-PreamblesPerSSB 指定可以映射到一个RO的SSB数量，以及可以映射到一个SSB的preamble的数量。

> 38.213-8.1:SS/PBCH block indexes provided by ssb-PositionsInBurst in SIB1 or in ServingCellConfigCommon are mapped to valid PRACH occasions in the following order where the parameters are described in [4, TS 38.211].
>-	First, in increasing order of preamble indexes within a single PRACH occasion
>-	Second, in increasing order of frequency resource indexes for frequency multiplexed PRACH occasions
>-	Third, in increasing order of time resource indexes for time multiplexed PRACH occasions within a PRACH slot
>-	Fourth, in increasing order of indexes for PRACH slots

## 举例
下图38.331 对于ssb-perRACH-OccasionAndCB-PreamblesPerSSB的定义
[![pkWUA74.jpg](https://s21.ax1x.com/2024/07/07/pkWUA74.jpg)](https://imgse.com/i/pkWUA74)

- **Example 01**

    - msg1-FDM = one<br/>
    - ssb-perRACH-OccasionAndCB-PreamblesPerSSB = one

[![pkWUu1x.png](https://s21.ax1x.com/2024/07/07/pkWUu1x.png)](https://imgse.com/i/pkWUu1x)

- **Example 02**

    - msg1-FDM = two<br/>
    - ssb-perRACH-OccasionAndCB-PreamblesPerSSB = one

[![pkWUKc6.png](https://s21.ax1x.com/2024/07/07/pkWUKc6.png)](https://imgse.com/i/pkWUKc6)

- **Example 03**

    - msg1-FDM = two<br/>
    - ssb-perRACH-OccasionAndCB-PreamblesPerSSB = eight

[![pkWUlnO.png](https://s21.ax1x.com/2024/07/07/pkWUlnO.png)](https://imgse.com/i/pkWUlnO)

- **Example 04**

    - msg1-FDM = two<br/>
    - ssb-perRACH-OccasionAndCB-PreamblesPerSSB = oneHalf

[![pkWU3He.png](https://s21.ax1x.com/2024/07/07/pkWU3He.png)](https://imgse.com/i/pkWU3He)

---

# Legacy RA

&emsp;&emsp;高层通过参数`ssb-perRACH-OccasionAndCB-PreamblesPerSSB`配置`N`个SSB关联一个PRACH occasion(参数`ssb-perRACH-Occasion`)，和每个SSB在每个有效PRACH occasion上基于竞争的preamble数(参数：`CB-preambles-per-SSB`)。其中对于N的配置有如下两种：

> 38.213-i30：
For Type-1 random access procedure, a UE is provided a number N of SS/PBCH block indexes associated with one PRACH occasion and a number R of contention based preambles per SS/PBCH block index per valid PRACH occasion by ssb-perRACH-OccasionAndCB-PreamblesPerSSB. 


[![pk2azz6.png](https://s21.ax1x.com/2024/07/03/pk2azz6.png)](https://imgse.com/i/pk2azz6)

[![pk2dotA.png](https://s21.ax1x.com/2024/07/03/pk2dotA.png)](https://imgse.com/i/pk2dotA)

说明
RACH参数配置中ssb-perRACH-OccasionAndCB-PreamblesPerSSB用于配置:
    - 每个RACH时刻对应的SSB个数N,从1/8-16
    - 当ssb-perRACH-Occasion>=1, N>=1时，即多个SSB对应1个RACH Occasion,从n*N<sup>total</sup><sub>preamble</sub>/N开始的连续CB PreamblesPerSSB个CB preambles对应于SSB n, 0<=n<=N-1 。

列举两个图片来说明RO与SSB的关系：

[![pkfcbsH.jpg](https://s21.ax1x.com/2024/07/09/pkfcbsH.jpg)](https://imgse.com/i/pkfcbsH)

[![pkfcXdI.jpg](https://s21.ax1x.com/2024/07/09/pkfcXdI.jpg)](https://imgse.com/i/pkfcXdI)

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

RACH partition为R17引入的新特性，旨在通过RACH流程，告知网络UE支持的特性（redCap，smallData，nsag，msg3-Repetitions）。

[![pkhnHIK.jpg](https://s21.ax1x.com/2024/07/10/pkhnHIK.jpg)](https://imgse.com/i/pkhnHIK)

- numberOfPreamblesPerSSB-ForThisPartition
    - It determines how many consecutive preambles are associated to the Feature Combination starting from the starting preamble(s) per SSB.


[![pkhuFJS.jpg](https://s21.ax1x.com/2024/07/10/pkhuFJS.jpg)](https://imgse.com/i/pkhuFJS)

> 38213 clause 8.1
[![pkhu3z4.jpg](https://s21.ax1x.com/2024/07/10/pkhu3z4.jpg)](https://imgse.com/i/pkhu3z4)


RACH partition有两个关键参数，分别为`numberOfPreamblesPerSSB-ForThisPartition` 和`startPreambleForThisPartition` ，其中`startPreambleForThisPartition` 标识了每个RACH partition的起始位置，`numberOfPreamblesPerSSB-ForThisPartition`标识了从这个起始位置开始，属于这个RACH partition的连续的P码个数。

## 一个SSB映射到多个RO

 对于一个SSB映射到多个RO的情况，即`N<1`的情况，在`totalNumberOfRA-Preambles`个P码对于发起RACH的SSB都可以使用，所以RACH partition的P码范围为`[startPreambleForThisPartition,startPreambleForThisPartition+numberOfPreamblesPerSSB-ForThisPartition)`

[![pkhMPgg.png](https://s21.ax1x.com/2024/07/10/pkhMPgg.png)](https://imgse.com/i/pkhMPgg)

## 多个SSB映射到一个RO

 对于多个SSB映射到一个RO的情况，即`N>=1`的情况,每个SSB其实只有N<sup>total</sup><sub>preamble</sub>/N个P码可以使用，其中CB preambles个数为开始的连续CB-PreamblesPerSSB，所以RACH partition的P码范围为`[n*Ntotalpreamble/N + startPreambleForThisPartition,n*Ntotalpreamble/N + startPreambleForThisPartition+numberOfPreamblesPerSSB-ForThisPartition)`

[![pkhKjHI.png](https://s21.ax1x.com/2024/07/10/pkhKjHI.png)](https://imgse.com/i/pkhKjHI)



---

# 参考文献

> [5g-nr-cell-scan-and-rach-procedure-poster](https://wirelessbrew.com/5g-nr/5g-nr-cell-scan-and-rach-procedure-poster/){:target="_blank"}

> [sharetechnote_5G_RACH](https://sharetechnote.com/html/5G/5G_RACH.html){:target="_blank"}

> [5G NR: 2-Step Random Access Procedure (Release-16)](https://howltestuffworks.blogspot.com/2020/04/5g-nr-2-step-random-access-procedure.html){:target="_blank"}

>  [ra_msg1](https://www.nrexplained.com/ra_msg1){:target="_blank"}