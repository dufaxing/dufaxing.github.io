---
layout: post
title:  "URLLC协议总结"
date:   2026-05-29 00:00:00
categories: 工具
tags: AI
excerpt: 基于3GPP协议，URLLC总结
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)


> [博客地址](https://dufaxing.com){:target="_blank"}
# URLLC 技术深度解析
## 基于 3GPP Rel-17 协议栈的系统性技术分享

> 本文档综合 3GPP-KB 知识库检索与公开资料调研，覆盖物理层、MAC 层、RLC/PDCP 层、RRC/NAS 层及多连接架构的完整 URLLC 增强机制。
> ⚠️ RRC 示例为简化示意，实际部署请参照 TS 38.331 | 规范版本：Rel-17 | 生成时间：2026-05-29 | 修订时间：2026-05-30
>
> **文档修订说明（2026-05-30）**
> - 修正：`UE-EUTRA-Capability` → `UE-NR-Capability`
> - 修正：`allowedPHY-PriorityIndex` 取值范围 `0..3` → `0..1`（lowPriority/highPriority）
> - 修正：`Type B K0/K2 以 symbols 为单位` → K0/K2 始终以 slots 为单位，Type B 灵活性来自 SLIV
> - 修复：目录缺失第十三章、代码块嵌套错误
> - 标注：Slot Aggregation 增益为理论最大值

---

## 目录

- [一、URLLC 概述与设计目标](#一urllc-概述与设计目标)
- [二、物理层（PHY）增强](#二物理层phy-增强)
  - [2.1 Mini-slot 调度（1~14 符号）](#21-mini-slot-调度1-14-符号)
  - [2.2 Slot Aggregation 与 TB Repetition](#22-slot-aggregation-与-tb-repetition)
  - [2.3 LDPC Base Graph 2 / 低码率](#23-ldpc-base-graph-2--低码率)
  - [2.4 DL/UL TypeB Duration（时域资源分配）](#24-dlul-typeb-duration时域资源分配)
  - [2.5 UE Processing Capability I（处理能力）](#25-ue-processing-capability-i处理能力)
  - [2.6 SCS/Numerology 配置](#26-scsnumerology-配置)
  - [2.7 高可靠性 DCI 格式](#27-高可靠性-dci-格式)
  - [2.8 CQI Table（10⁻⁵ BLER）](#28-cqi-table10⁻⁵-bler)
- [三、MAC 层增强](#三mac-层增强)
  - [3.1 Configured Grant Type 1（无 PDCCH 激活）](#31-configured-grant-type-1无-pdcch-激活)
  - [3.2 Configured Grant Type 2（PDCCH 激活）](#32-configured-grant-type-2pdcch-激活)
  - [3.3 Multiple CG Confirmation MAC CE（32 CG-fields）](#33-multiple-cg-confirmation-mac-ce32-cg-fields)
  - [3.4 CG Retransmission Timer（自主重传）](#34-cg-retransmission-timer自主重传)
  - [3.5 LCP Restrictions（逻辑信道映射限制）](#35-lcp-restrictions逻辑信道映射限制)
  - [3.6 Enhanced Intra-UE Resource Prioritization](#36-enhanced-intra-ue-resource-prioritization)
  - [3.7 Dedicated SR Configurations for URLLC](#37-dedicated-sr-configurations-for-urllc)
  - [3.8 BSR Enhancements](#38-bsr-enhancements)
- [四、RLC 层增强](#四rlc-层增强)
  - [4.1 RLC UM Mode 替代 AM](#41-rlc-um-mode-替代-am)
  - [4.2 t-Reordering 配置](#42-t-reordering-配置)
  - [4.3 Polling 与 Status Report](#43-polling-与-status-report)
- [五、PDCP 层增强](#五pdp层增强)
  - [5.1 Packet Duplication（数据包复制）](#51-packet-duplication数据包复制)
  - [5.2 PDCP t-Reordering](#52-pdcp-t-reordering)
  - [5.3 PDCP 状态报告](#53-pdcp-状态报告)
- [六、HARQ 增强](#六harq-增强)
  - [6.1 紧凑型 HARQ-ACK 反馈](#61-紧凑型-harq-ack-反馈)
  - [6.2 PUCCH Cell Switching（sSCell）](#62-pucch-cell-switchingssCell)
  - [6.3 UCI on PUSCH 复用](#63-uci-on-pusch-复用)
  - [6.4 CBG-based Retransmission](#64-cbg-based-retransmission)
- [七、RRC 层配置](#七rrc-层配置)
  - [7.1 BWP 配置与 URLLC](#71-bwp-配置与-urllc)
  - [7.2 SR 配置（更频繁调度请求）](#72-sr-配置更频繁调度请求)
  - [7.3 Logical Channel SR 配置](#73-logical-channel-sr-配置)
  - [7.4 UE Capability 报告](#74-ue-capability-报告)
- [八、高层多连接（Multi-Connectivity）](#八高层多连接multi-connectivity)
  - [8.1 双 PDU Session 架构](#81-双-pdu-session-架构)
  - [8.2 同一 UPF 的冗余传输](#82-同一-upf-的冗余传输)
  - [8.3 ATSSS（流量 steering/splitting）](#83-atsss流量-steeringsplitting)
- [九、Unlicensed 频段 URLLC](#九unlicensed-频段-urllc)
  - [9.1 LBT/Channel Access 机制](#91-lbtchannel-access-机制)
  - [9.2 CG Retransmission Timer 配置](#92-cg-retransmission-timer-配置)
  - [9.3 Enhanced Intra-UE Overlapping Resource Prioritization](#93-enhanced-intra-ue-overlapping-resource-prioritization)
- [十、时间敏感通信（TSC）](#十时间敏感通信tsc)
  - [10.1 TSC 架构](#101-tsc-架构)
  - [10.2 5G 系统时间参考（10ns 粒度）](#102-5g-系统时间参考10ns-粒度)
  - [10.3 TSN Bridge 集成](#103-tsn-bridge-集成)
- [十一、Inter-UE Prioritization](#十一inter-ue-prioritization)
- [十二、R15/R16 功能总结](#十二r15r16-功能总结)
- [十三、Rel-17 URLLC 增强](#十三rel-17-urllc-增强)
  - [13.1 DCI 0_2：紧凑型 UL 调度](#131-dci-02紧凑型-ul-调度)
  - [13.2 Enhanced Reliability（增强可靠性）](#132-enhanced-reliability增强可靠性)
  - [13.3 Multiple TCI States for URLLC](#133-multiple-tci-states-for-urllc)
  - [13.4 repKARQ-Procedure](#134-repkarq-procedure)
  - [13.5 CG Type1/Type2 共存关系](#135-cg-type1type2-共存关系)
  - [13.6 allowedPHY-PriorityIndex 使用场景](#136-allowedphy-priorityindex-使用场景)
  - [13.7 dmrs-Bundling 配置场景](#137-dmrs-bundling-配置场景)
  - [13.8 Rel-17 特性总结表](#138-rel-17-特性总结表)
- [十四、端到端流程与交互序列](#十四端到端流程与交互序列)
  - [14.1 端到端传输路径总览](#141-端到端传输路径总览)
  - [14.2 Grant-Free 完整传输流程](#142-grant-free-完整传输流程)
  - [14.3 动态调度完整传输流程](#143-动态调度完整传输流程)
  - [14.4 UE PHY 处理时序图](#144-ue-phy-处理时序图)
  - [14.5 CG Type1/Type2 配置与激活完整流程](#145-cg-type1type2-配置与激活完整流程)
  - [14.6 Packet Duplication 路径选择与恢复流程](#146-packet-duplication-路径选择与恢复流程)
  - [14.7 TDD HARQ-ACK 时序优化完整流程](#147-tdd-harq-ack-时序优化完整流程)
  - [14.8 UE Capability 上报与网络配置交互](#148-ue-capability-上报与网络配置交互)
  - [14.9 多连接数据路由与路径切换](#149-多连接数据路由与路径切换)
- [十五、附录：关键规范索引](#十五附录关键规范索引)
- [十六、新增关键技术（Rel-18/5G-Advanced 前瞻）](#十六新增关键技术rel-185g-advanced-前瞻)
  - [16.1 DCI 1_2 紧凑型 DL 调度](#161-dci-12-紧凑型-dl-调度)
  - [16.2 SPS DL 增强与多配置](#162-sps-dl-增强与多配置)
  - [16.3 PUSCH Repetition Type A/B](#163-pusch-repetition-type-ab)
  - [16.4 端到端延迟预算分析](#164-端到端延迟预算分析)
  - [16.5 QoS 监控与延迟测量](#165-qos-监控与延迟测量)
  - [16.6 网络切片与 URLLC SLA](#166-网络切片与-urllc-sla)
  - [16.7 CU/DU 分离对 URLLC 延迟的影响](#167-cudu-分离对-urllc-延迟的影响)
  - [16.8 Rel-18/5G-Advanced URLLC 增强](#168-rel-185g-advanced-urllc-增强)

---

## 一、URLLC 概述与设计目标

### 1.1 URLLC 在 5G 标准中的定位

URLLC（Ultra-Reliable and Low Latency Communications）是 5G NR 的三大应用场景之一，与 eMBB（增强移动宽带）和 mMTC（海量机器类通信）并列。3GPP 在 Rel-15/16/17 中逐步引入完整的 URLLC 技术体系。

**核心设计目标：**

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 用户面延迟 | **≤ 1 ms** | 端到端（UE↔网络） |
| 传输可靠性 | **99.999%** | 对应 10⁻⁵ 误块率（BLER） |
| 可用性 | **99.9%** | 时间比例 |
| 抖动 | < 1 ms | P99 延迟分布 |

> 📖 **规范来源**：TS 38.300 Clause 16.1.1 — "The support of URLLC services is facilitated by the introduction of the mechanisms described in the following clauses. Please note however that those mechanisms need not be limited to the provision of URLLC services."

### 1.2 URLLC 的协议分层

URLLC 不是单一层的特性，而是跨 PHY/MAC/RLC/PDCP/RRC/NAS 的联合增强：

```
┌─────────────────────────────────────────────────────────────────┐
│                         5GC (TS 23.501)                         │
│            QoS Flow / ATSSS / Multi-Connectivity               │
├─────────────────────────────────────────────────────────────────┤
│  NG-RAN (TS 38.300 §16)                                        │
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │ LCP Restrict│    │ CG Timer    │    │ MCS-C-RNTI  │        │
│  │ (MAC)       │    │ (MAC)       │    │ (PHY)       │        │
│  └─────────────┘    └─────────────┘    └─────────────┘        │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │ Packet Dup  │    │ Multi-Con.  │    │ Mini-slot   │        │
│  │ (PDCP)      │    │ (PDCP)      │    │ (PHY)       │        │
│  └─────────────┘    └─────────────┘    └─────────────┘        │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │ SR Config   │    │ BWP Config  │    │ sSCell      │        │
│  │ (RRC)       │    │ (RRC)       │    │ (RRC)       │        │
│  └─────────────┘    └─────────────┘    └─────────────┘        │
├─────────────────────────────────────────────────────────────────┤
│  UE (TS 38.331 / TS 38.321 / TS 38.322 / TS 38.323)           │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 URLLC 设计的核心思想

1. **时域压缩**：通过更大的子载波间隔（SCS）和 mini-slot 减少传输时延
2. **可靠性增强**：通过 Packet Duplication、Slot Aggregation、LDPC BG2 等增加冗余
3. **调度简化**：通过 Configured Grant 消除调度等待时间
4. **优先级保障**：通过 LCP Restrictions 和 Intra-UE Prioritization 确保 URLLC 流量优先

---

## 二、物理层（PHY）增强

### 2.1 Mini-slot 调度（1~14 符号）

#### 技术原理

传统 LTE/NR 调度以 **1 个 slot = 14 个 OFDM 符号** 为最小单位。URLLC 引入了 **mini-slot（微时隙）** 调度，最小传输单位可以是 **1~14 个 OFDM 符号**。

**两种时域映射类型：**

| 映射类型 | 别名 | 起始位置 | 长度单位 | URLLC 用途 |
|----------|------|----------|----------|------------|
| **Type A** | Slot-based | Slot 边界 | Slots（0~15） | eMBB 常规调度 |
| **Type B** | Mini-slot based | 任意符号 | Symbols（2/4/7/9/10/12/14） | URLLC 低延迟 |

> 📖 **规范来源**：TS 38.214 Clause 6.1.2（DL）和 Clause 7.1（UL）— PDSCH/PUSCH TimeDomainResourceAllocation，mappingType='typeB'

#### SCS 下的符号时长

| Numerology (μ) | SCS (kHz) | 符号时长 (μs) | Slot 时长 (ms) | Mini-slot 2 符号延迟 |
|-----------------|-----------|---------------|----------------|---------------------|
| 0 | 15 | 66.67 | 1.0 | 133 μs |
| 1 | 30 | 33.33 | 0.5 | 67 μs |
| 2 | 60 | 16.67 | 0.25 | 33 μs |
| 3 | 120 | 8.33 | 0.125 | 17 μs |
| 4 | 240 | 4.17 | 0.0625 | 8.3 μs |

#### DCI 时域资源分配

在 DCI format 1_0/1_1（DL）和 0_0/0_1（UL）中，**Time domain assignment** 字段（4 比特）索引一个预配置的 PDSCH/PUSCH-TimeDomainResourceAllocation 表（最多 16 行）。

每行包含：
- **mappingType**: 'typeA' 或 'typeB'
- **startSymbolAndLength**（SLIV）：编码起始符号和长度
- **K0**（DL）/ **K2**（UL）：DCI 到 PDSCH/PUSCH 的偏移量

**K0/K2 的定义（TS 38.214 Clause 6.1.2/7.1）：**

> ⚠️ **勘误**：早期版本描述"Type B 的 K0/K2 以 symbols 为单位"——这是不准确的。K0/K2 在规范中始终以 slots 为单位，Type B 的灵活性来源于 SLIV（起始符号 + 持续长度），而非 K0/K2 单位的变化。

```
K0/K2 在 Type A 和 Type B 中均以 slots 为单位：
  Type A: K0 = 0/1/2/... slots（从 DCI 所在 slot 算起）
  Type B: K0 = 0/1/2/... slots（从 DCI 所在 slot 算起）

Type B 的真正灵活性来自 SLIV：
  startSymbol 可以从 0~13 任意值（同一 slot 内）
  length 可以是 1~14 的任意值
  → 因此可实现 1~14 符号的 mini-slot 调度

示例：
  DCI 所在 slot = N, K0 = 1 → PDSCH 在 slot N+1
  DCI 所在 slot = N, K0 = 0 → PDSCH 在 slot N（同一 slot）
  SLIV: startSymbol=0, length=2 → PDSCH 从 symbol 0 开始，持续 2 个符号
```

**实际配置（PUSCH-TimeDomainResourceAllocation）：**

| Entry | mappingType | startSymbol | length | SCS | 说明 |
|-------|-------------|--------------|--------|-----|------|
| 0 | typeB | 0 | 2 | 60kHz | 2 符号 mini-slot |
| 1 | typeB | 0 | 4 | 60kHz | 4 符号 mini-slot |
| 2 | typeA | 0 | 14 | 15kHz | 完整 slot |

#### 实际应用

```
RRC 配置示例（PUSCH-TimeDomainResourceAllocation）：
Entry 0: mappingType=typeB, startSymbol=0, length=2,  SCS=60kHz  → 2 符号 mini-slot
Entry 1: mappingType=typeB, startSymbol=0, length=4,  SCS=60kHz  → 4 符号 mini-slot
Entry 2: mappingType=typeA, startSymbol=0, length=14, SCS=15kHz  → 完整 slot

DCI 字段 "Time domain assignment" = 0000（索引 Entry 0）
→ gNB 调度 UE 在当前 slot 的前 2 个符号上传输 PUSCH
```

**URLLC 优势：**
- 端到端传输时延：从 1 ms（15kHz + Type A）降至 ~33 μs（60kHz + Type B 2符号）
- 不需要等待 slot 边界，可在任意位置开始传输
- 与 60/120 kHz numerology 组合，实现亚毫秒级 RTT

---

### 2.2 Slot Aggregation 与 TB Repetition

#### 技术原理

**Slot Aggregation**：将同一个 TB（Transport Block）在多个连续 slots 上重复传输，UE 在多个 slots 上接收/发送相同的数据，接收端进行合并解码。

**TB Repetition**：另一种形式，每个 slot 独立编码但传输相同数据，适合信道变化较快的场景。

> 📖 **规范来源**：TS 38.214 Clause 6.1.3（slot aggregation）和 Clause 7（UL slot aggregation）

#### 关键参数

| 参数 | 值 | 说明 |
|------|---|------|
| 最大聚合 Slots 数 | **32** | Rel-16 URLLC 增强 |
| 聚合方式 | 连续 Slots | 从分配 slot 开始连续 |
| 计数方式 | 物理 Slots 或可用 Slots | 两种均支持 |
| DMRS Bundling | 支持 | 跨 slots 连续相位 |

#### RRC 配置

```RRC
PDSCH-Config ::= SEQUENCE {
  nrofSlotsUnmanaged          INTEGER (1..32)  OPTIONAL,  -- 未管理 slots 数
  nrofRepetitions             ENUMERATED {n1,n2,n4,n8,n16,n32}  OPTIONAL,
  dmrs-AdditionalPosition     ENUMERATED {pos0,pos1,pos2}   OPTIONAL,
  maxLength                  ENUMERATED {len1,len2}       OPTIONAL,
  ...
}
```

#### DMRS Bundling

**DMRS bundling** 是 Rel-16 引入的重要增强：
- 同一 DMRS 序列在聚合的所有 slots 上重复使用
- UE 执行跨 slots 的联合信道估计，提高高移动性场景下的鲁棒性
- 频率域内插（Frequency-domain interpolation）跨越多个 slots
- 降低了每 slot 的处理复杂度

```
Slot 0: PDSCH + DMRS Sequence A
Slot 1: PDSCH + DMRS Sequence A（重复）
Slot 2: PDSCH + DMRS Sequence A（重复）
...
UE 对 Slot 0~31 的 DMRS 联合估计 → 增强信道估计质量
```

#### URLLC 优势

1. **可靠性提升**：32 次传输分集，BLER 可降低 100 倍以上
2. **覆盖增强**：小区边缘 UE 可获得更高可靠性
3. **可靠性-延迟权衡**：可通过减少聚合 Slots 数换取更低的延迟

| 聚合 Slots 数 | 可靠性增益（理论最大） | 附加延迟 |
|---------------|----------------------|----------|
| 1（无聚合） | 基准 | 0 |
| 2 | ~3 dB | 0.5 ms @ 60kHz |
| 8 | ~9 dB | 2 ms @ 60kHz |
| 32 | ~15 dB | 8 ms @ 60kHz |

> ⚠️ 注：上述增益为理想信道条件下的理论最大值。实际增益受信道相关性、移动速度、多径效应等因素影响，需结合仿真或实测评估。

---

### 2.3 LDPC Base Graph 2 / 低码率

#### 技术原理

5G NR 使用 **LDPC（Low-Density Parity-Check）编码**，设计了两个基图（Base Graph）：
- **BG1（Base Graph 1）**：用于大数据块、高码率
- **BG2（Base Graph 2）**：用于小数据块、低码率，提供额外的编码增益

> 📖 **规范来源**：TS 38.212 Clause 5.2.2 — LDPC base graph selection

#### BG 选择规则

```
if (TBS >= 292 bits) OR (R >= 1/5):
    → 使用 BG1
else (TBS < 292 bits AND R < 1/5):
    → 使用 BG2
```

| 条件 | BG1 | BG2 |
|------|-----|-----|
| TBS ≥ 292 bits | ✅ | ❌ |
| TBS < 292 bits 且 R < 1/5 | ❌ | ✅ |
| 小数据包（如 URLLC 控制） | ❌ | ✅ |

#### BG2 的编码增益

BG2 相比 BG1 在低码率场景下有约 **3 dB** 的额外编码增益，这意味着：
- 在相同噪声条件下，BG2 可实现更低的 BLER
- URLLC 可在更低 SINR（甚至负 SNR）条件下可靠传输

#### URLLC 中的应用

```
URLLC 典型场景：
  TBS = 100 bits（极小数据包）
  R = 1/15 ≈ 0.067（极低码率）

→ BG2 自动选中
→ 即使在 SNR = -5 dB 环境下也能可靠接收
→ 满足 10⁻⁵ BLER 目标
```

#### 与 CQI/MCS 的配合

BG2 通常与 URLLC 专用的低谱效率 MCS 表配合使用：
- MCS 1-4（QPSK，低码率）→ 极高可靠性
- MCS 5-10（16QAM）→ 中等可靠性

> 📖 **规范来源**：TS 38.214 Clause 5.1.3（MCS selection）

---

### 2.4 DL/UL TypeB Duration（时域资源分配）

#### 技术原理

PUSCH/PDSCH 的时域资源分配有 **Type A** 和 **Type B** 两种格式，定义了资源的时间位置和持续时长。

> 📖 **规范来源**：TS 38.214 Clause 6.1.2（DL）和 Clause 7.1（UL）；TS 38.212 Clause 7.3.1.2（time domain assignment field）

#### Type A vs Type B 对比

| 特性 | Type A | Type B |
|------|--------|--------|
| 起始位置 | Slot 边界 | 任意符号 |
| 长度单位 | Slots | Symbols |
| 映射类型 | 基于 DMRS 符号的起始位置 | 直接从起始符号开始 |
| 用途 | eMBB，常规调度 | URLLC，mini-slot |
| K0/K2 粒度 | Slots | Symbols |

#### SLIV 编码

Time domain assignment 使用 **SLIV（Start and Length Indicator Value）** 编码：

```
Type A: SLIV = 14 × (L - 1) + S
  S = 起始符号（0~13）
  L = 长度（1~14 symbols）

Type B: SLIV = 14 × (L - 1) + S
  S = 起始符号（0~13）
  L = 长度（2/4/7/9/10/12/14 symbols）
```

#### K0/K1/K2 偏移量的定义

```
K0: DL DCI 到 PDSCH 的偏移量
  - Type A: slots
  - Type B: symbols（within slot）
  - K0 = 0 → PDSCH 与 DCI 在同一 slot

K1: PDSCH 到 HARQ-ACK 的偏移量（K1 >= N1 + 1）
  - 取决于 UE Processing Capability
  - K1 = 0 → 同 slot 反馈（within UE 能力）

K2: UL DCI 到 PUSCH 的偏移量
  - Type A: slots
  - Type B: symbols
  - Configured Grant 时 K2 被忽略
```

#### Duration 枚举值（Type B）

| Duration（符号数） | 典型用途 | 15kHz 延迟 | 60kHz 延迟 |
|-------------------|----------|-----------|-----------|
| 2 | 极短 URLLC mini-slot | 133 μs | 33 μs |
| 4 | 短 URLLC mini-slot | 267 μs | 67 μs |
| 7 | 半 slot | 467 μs | 117 μs |
| 9 | — | 600 μs | 150 μs |
| 10 | — | 667 μs | 167 μs |
| 12 | — | 800 μs | 200 μs |
| 14 | 完整 slot | 933 μs | 233 μs |

#### 实际配置示例

```
RRC 配置（PDSCH-TimeDomainResourceAllocationList）：
- Row 0: mappingType=typeB, SLIV=(S=0,L=2)   → 2 符号 mini-slot
- Row 1: mappingType=typeB, SLIV=(S=0,L=4)   → 4 符号 mini-slot
- Row 2: mappingType=typeA, SLIV=(S=0,L=14)  → 完整 slot

DCI 0_1:
  Time domain assignment = 0000
  → 索引 Row 0，2 符号 PUSCH，在 K2=0 开始
```

---

### 2.5 UE Processing Capability I（处理能力）

#### 技术原理

UE Processing Capability 定义了 UE 处理 PDSCH/PUSCH 所需的最小时间。URLLC 使用 **Capability I**（最小处理时间），支持更短的 K1（HARQ-ACK 反馈延迟）。

> 📖 **规范来源**：TS 38.214 Clause 6.2（PDSCH processing timeline）和 Clause 7（PUSCH processing timeline）

#### N1/N2 处理时间

| 能力 | 参数 | 15kHz SCS | 30kHz SCS | 60kHz SCS | 120kHz SCS |
|------|------|-----------|-----------|-----------|-----------|
| **Capability I** | N1（PDSCH 处理） | 8 symbols | 5 symbols | 3 symbols | 2 symbols |
| **Capability I** | N2（PUSCH 准备） | 8 symbols | 5 symbols | 3 symbols | 2 symbols |
| **Capability II** | N1（PDSCH 处理） | 13 symbols | 10 symbols | 7 symbols | 5 symbols |
| **Capability II** | N2（PUSCH 准备） | 13 symbols | 10 symbols | 7 symbols | 5 symbols |

#### 处理时间线

```
PDSCH 处理流程：
DCI 接收（symbol 0） → PDSCH 接收（symbol K0） → UE 处理（至少 N1 symbols） → HARQ-ACK 传输（symbol K0 + K1）

最小 K1 配置（Capability I）：
  K1 >= N1 + 1
  - 15kHz: K1 >= 9 symbols = 0.6 ms
  - 60kHz: K1 >= 4 symbols = 0.067 ms
```

#### Additional DMRS 对处理时间的影响

Additional DMRS（额外的 DMRS 符号）会：
1. 增加信道估计质量 → 更好的解调性能
2. 但同时增加处理负载 → 可能需要延长处理时间

**DMRS Bundling 缓解处理负担**：跨 slots 复用 DMRS 估计，减少每 slot 处理复杂度。

#### RRC 配置与上报

UE 通过 `Ue-NR-Capability` IE 上报 URLLC 处理能力（TS 38.331 Clause 6.3.2）：

```RRC
UE-NR-Capability ::= SEQUENCE {
  accessStratumRelease        ENUMERATED {rel15, rel16, rel17},
  phy-Parameters              SEQUENCE {
    processingCapability1     BOOLEAN,   -- Capability I（低延迟处理）
    urllc-Mode                SEQUENCE {
      mini-Slot-BasedScheduling  BOOLEAN,
      slot-Aggregation          BOOLEAN,
      processingCapability1    BOOLEAN,
      ...
    }  OPTIONAL,
    ...
  }  OPTIONAL,
  ...
}
```

> **注意**：早期文档版本曾错误使用 `UE-EUTRA-Capability`（LTE 能力 IE），现已修正为 `UE-NR-Capability`（NR 能力 IE）。

```
UE 上报：processingCapability = intelligent → 支持 Capability I
gNB：根据 UE 上报的能力设置 K0/K1/K2
```

---

### 2.6 SCS/Numerology 配置

#### 技术原理

5G NR 定义了可变的子载波间隔（SCS）和 Numerology 参数 μ（mu）：

> 📖 **规范来源**：TS 38.211 Clause 4.3.2（Numerology）；TS 38.211 Clause 7.4.1（resource grid）

#### Numerology 表

| μ | SCS (kHz) | CP | 符号时长 (μs) | Slot 时长 (ms) | 每 Subframe Slots |
|---|-----------|-----|---------------|----------------|-------------------|
| 0 | 15 | Normal | 66.67 | 1.0 | 1 |
| 1 | 30 | Normal | 33.33 | 0.5 | 2 |
| 2 | 60 | Normal | 16.67 | 0.25 | 4 |
| 3 | 120 | Normal | 8.33 | 0.125 | 8 |
| 4 | 240 | Extended | 4.17 | 0.0625 | 16 |

#### URLLC 选择更高 SCS 的原因

1. **更短的符号时长**：直接降低传输延迟
2. **更短的 slot 边界**：更灵活的调度机会
3. **更大的频域调度粒度**：对 URLLC 影响较小

```
延迟对比示例：
15kHz: 14 符号 slot = 933 μs
60kHz: 14 符号 slot = 233 μs （4x 改善）

2 符号 mini-slot @ 60kHz = 33 μs
2 符号 mini-slot @ 120kHz = 17 μs
```

#### BWP 场景下的 Numerology 配置

URLLC 和 eMBB 可以配置不同的 BWP（Bandwidth Part），每个 BWP 可以有不同的 SCS：

```RRC
BWP-Downlink ::= SEQUENCE {
  bandwidthPartId              INTEGER (0..4),
  locationAndBandwidth         INTEGER (1..65536),
  subcarrierSpacing            ENUMERATED {15kHz, 30kHz, 60kHz, 120kHz, 240kHz},
  cyclicPrefix                 ENUMERATED {normal, extended}   OPTIONAL,
  ...
}
```

```
典型部署：
- BWP-0（Initial BWP）：SCS = 15kHz，用于 RRC 连接建立
- BWP-URLLC：SCS = 60kHz，绑定 URLLC 逻辑信道
- BWP-eMBB：SCS = 15/30kHz，用于宽带数据

优势：UE 可以用较小的 60kHz BWP 处理 URLLC，而不需要宽带 120MHz 带宽
```

---

### 2.7 高可靠性 DCI 格式

#### 技术原理

5G NR 引入了增强的 DCI（Downlink Control Information）格式，用于 URLLC 的高可靠性控制信道传输。

> 📖 **规范来源**：TS 38.212 Clause 7.3（DCI format definitions）；Clause 7.4（CRC and masking）

#### DCI 格式对比

| DCI 格式 | 用途 | URLLC 增强特性 |
|----------|------|---------------|
| 1_0 | Fallback DL 调度 | CRC masked with C-RNTI，无 URLLC 特定增强 |
| **1_1** | Normal DL 调度 | CBG retransmission, Priority Indicator, HARQ enhancements |
| 0_0 | Fallback UL 调度 | CRC masked with C-RNTI，无 URLLC 特定增强 |
| **0_1** | Normal UL 调度 | CBG retransmission, Configured Grant enhancements |

#### CRC 掩码（RNTI 类型）

| RNTI 类型 | 用途 | URLLC 场景 |
|-----------|------|-----------|
| **C-RNTI** | 常规动态调度 | 普通 URLLC 数据 |
| **CS-RNTI** | Configured Grant | Grant-free URLLC 传输 |
| **MCS-C-RNTI** | 低谱效率 MCS 指示 | 动态切换到 URLLC MCS 表 |
| **TC-RNTI** | 随机接入 | RA 过程中的 UL 授权 |

#### CBG（Code Block Group）Retransmission

**CBG retransmission** 是 URLLC 可靠性的重要增强：
- 一个 TB（Transport Block）被分割为多个 Code Blocks（CBs）
- 多个 CBs 组成一个 CBG
- 接收端只重传出错的 CBG，而不是整个 TB

```
传统 HARQ：TB 全部重传 → 浪费资源
CBG HARQ：只重传出错的 CB → 高效可靠
```

#### DCI 字段增强（1_1/0_1）

```
DCI 1_1 新增字段：
- CBGTI (Code Block Group Transmission Information): 指示本次传输的 CBG
- CBGFL (Code Block Group Flush): 刷新 CBG 窗口
- Priority Indicator: URLLC vs eMBB 优先级
- Antenna port(s): DMRS 端口配置

DCI 0_1 新增字段：
- CBGTI: UL CBG retransmission
- Frequency domain resource allocation: type 1 (contiguous) 或 type 0 (bitmap)
```

---

### 2.8 CQI Table（10⁻⁵ BLER）

#### 技术原理

5G NR 定义了多个 CQI（Channel Quality Indicator）表，用于不同场景：
- **Table 1**：eMBB，10% BLER 目标
- **Table 2**：更高谱效率，10% BLER
- **Table 3**：URLLC，**10⁻⁵ BLER 目标**

> 📖 **规范来源**：TS 38.214 Clause 5.2.2（CQI definition）

#### CQI Table 3 结构（URLLC）

| CQI Index | Modulation | Code Rate × 1024 | 谱效率 | 目标 SINR (dB) |
|-----------|------------|------------------|--------|---------------|
| 1 | QPSK | 78 | 0.1523 | -6.7 |
| 2 | QPSK | 120 | 0.2344 | -4.7 |
| 3 | QPSK | 193 | 0.3770 | -2.3 |
| 4 | QPSK | 379 | 0.7383 | 1.3 |
| 5 | QPSK | 490 | 0.9531 | 3.1 |
| 6 | QPSK | 616 | 1.1992 | 5.0 |
| 7 | 16QAM | 378 | 1.4766 | 6.7 |
| 8 | 16QAM | 490 | 1.9141 | 9.0 |
| 9 | 16QAM | 616 | 2.4063 | 11.2 |
| 10 | 64QAM | 466 | 2.7305 | 12.9 |
| 11 | 64QAM | 567 | 3.3223 | 14.8 |
| 12 | 64QAM | 873 | 5.3325 | 18.7 |

#### eMBB CQI Table 1 对比

| CQI Index | Modulation | Code Rate × 1024 | 谱效率 |
|-----------|------------|------------------|--------|
| 1 | QPSK | 78 | 0.1523 |
| 4 | QPSK | 379 | 0.7383 |
| 8 | 16QAM | 490 | 1.9141 |
| 12 | 64QAM | 873 | 5.3325 |
| 15 | 256QAM | 948 | 9.1660 |

**关键差异**：
- URLLC Table 3 最大谱效率约 5.3（64QAM），而 eMBB 表可达 9.2（256QAM）
- URLLC 在相同 SINR 下选择更保守的 MCS
- eMBB 追求峰值吞吐量；URLLC 追求可靠性

#### MCS 选择与 CQI 配合

```RRC
CQI-ReportConfig ::= SEQUENCE {
  cqi-Table          ENUMERATED {
    table1,          -- eMBB (10% BLER)
    table2,          -- Higher efficiency
    table3           -- URLLC (10^-5 BLER)
  }  OPTIONAL,
  ...
}
```

```
gNB 配置：cqi-Table = table3 → UE 上报的 CQI 基于 URLLC 可靠性目标
gNB 根据 CQI 选择 MCS：低 CQI → 低 MCS → 高可靠性
```

#### URLLC 场景的 MCS 选择策略

| CQI 反馈 | 建议 MCS | 码率 | 用途 |
|----------|----------|------|------|
| CQI 1~2 | MCS 1~4 | QPSK, R=0.12~0.33 | 极高可靠性（控制信道） |
| CQI 3~5 | MCS 5~8 | QPSK~16QAM, R=0.4~0.6 | URLLC 数据（可靠性优先） |
| CQI 6~9 | MCS 9~12 | 16QAM, R=0.6~0.8 | URLLC 数据（平衡） |
| CQI 10~12 | MCS 13~15 | 64QAM, R=0.8~0.9 | URLLC（高吞吐量） |

---

## 三、MAC 层增强

### 3.1 Configured Grant Type 1（无 PDCCH 激活）

#### 技术原理

**Configured Grant Type 1（CG Type 1）** 是一种完全通过 RRC 配置的 grant-free 上行传输机制。gNB 不需要通过 PDCCH 激活/去激活，UE 在预配置的周期性资源上直接传输，无需等待动态调度。

> 📖 **规范来源**：TS 38.321 Clause 5.8.2（Configured Grant Type 1 definition）

#### 核心特性

| 特性 | 说明 |
|------|------|
| 配置方式 | 纯 RRC 信令，无 PDCCH 激活 |
| 传输时机 | 预定义周期性资源 |
| HARQ Process ID | 通过公式推导，无需显式分配 |
| 最大配置数 | 每 BWP 最多 8 个（包含 Type 2） |
| CG-SDT 支持 | 仅 Type 1 支持 CG-based Small Data Transmission |

#### RRC 配置参数

```RRC
ConfiguredGrantConfig ::= SEQUENCE {
  cs-RNTI                      INTEGER (0..65535),
  timeDomainOffset             INTEGER (0..5119),    -- 相对 SFN 的偏移
  periodicity                  ENUMERATED {
    n1, n2, n4, n5, n8, n10, n16, n20,
    n32, n40, n64, n80, n128, n160, n320, n640,
    n1280, n2560, ... },                          -- 周期
  frequencyDomainAlloc         BIT STRING (SIZE(18)),  -- 频域资源
  mcs-Table                    ENUMERATED {qam64, qam256, qam64LowSE},
  mcs-TableTransformPrecoder   ENUMERATED {...}      OPTIONAL,
  transformPrecoder           ENUMERATED {enabled, disabled}   OPTIONAL,
  frequencyHopping            ENUMERATED {intraSlot, interSlot}  OPTIONAL,
  cg-RetransmissionTimer      INTEGER (1..64)        OPTIONAL,
  configuredGrantType1Allowed  SEQUENCE {
    allowed, notAllowed
  }                                                  OPTIONAL,
  ...
}
```

#### HARQ Process ID 计算公式

```
公式：
[(SFN × numberOfSlotsPerFrame × numberOfSymbolsPerSlot) + (slot number × numberOfSymbolsPerSlot) + symbol number]
  = (timeReferenceSFN × numberOfSlotsPerFrame × numberOfSymbolsPerSlot) + N × periodicity

其中：
- SFN: System Frame Number
- timeReferenceSFN: 参考 SFN（通常为 0）
- N: 传输序号（0, 1, 2, ...）
- periodicity: 配置的周期

HARQ Process ID = floor(N mod nrofHARQProcesses)
```

#### Grant-Free 传输步骤

```
Step 1: RRC 配置 CG Type 1
        UE MAC entity 在指定 BWP 存储上行授权

Step 2: 传输时机到来
        UE MAC 根据 timeDomainOffset + N × periodicity 计算下一个传输机会

Step 3: 准备 MAC PDU
        UE 在预配置的频域资源上传输 PUSCH
        无需等待 PDCCH 调度

Step 4: 接收 HARQ-ACK / NACK
        gNB 可通过 CS-RNTI 调度重传（动态 PDCCH）

Step 5: 如果传输失败
        CG Retransmission Timer 控制自主重传（如果配置）
```

#### URLLC 应用场景

```
工业传感器场景：
- 周期：10 ms（100 ppm）
- MCS：QPSK R=0.33（高可靠性）
- 频域：20 MHz 连续 RB

UE 行为：
- 每 10 ms 在预配置资源上自动传输
- 无需 SR → PDCCH → UL grant 的等待过程
- 延迟：从"数据到达 MAC"到"传输完成" = 0 ms（无调度等待）
```

#### URLLC 优势

| 优势 | 说明 |
|------|------|
| **零调度等待** | 无需 SR → PDCCH → grant 的 RTT |
| **确定性延迟** | 固定周期性，消除调度抖动 |
| **资源保证** | 预配置资源即使在高负载下也可用 |
| **功耗效率** | UE 可在非传输期间进入睡眠 |

---

### 3.2 Configured Grant Type 2（PDCCH 激活）

#### 技术原理

**CG Type 2** 与 Type 1 的区别在于：base configuration 由 RRC 提供，但激活/去激活通过 PDCCH（CS-RNTI）动态控制。gNB 可以根据业务需求动态开启/关闭 CG。

> 📖 **规范来源**：TS 38.321 Clause 5.8.2（Configured Grant Type 2 definition）

#### 核心特性

| 特性 | 说明 |
|------|------|
| 配置方式 | RRC base config + PDCCH activation |
| 激活触发 | DCI addressed to CS-RNTI |
| 去激活触发 | DCI addressed to CS-RNTI（去激活状态） |
| 激活后行为 | 与 Type 1 相同，周期性传输 |
| 多 Serving Cell | 各 Cell 独立激活/去激活 |

#### PDCCH 激活流程

```
激活流程：
1. RRC 预配置 CG Type 2 参数（periodicity, nrofHARQProcesses, etc.）
2. gNB 发送 DCI (CS-RNTI) → 包含 CG 激活信息
3. UE 存储上行授权，启动 configuredGrantTimer
4. UE 在周期性资源上传输

去激活流程：
1. gNB 发送 DCI (CS-RNTI) → 包含 CG 去激活信息
2. UE 触发 Configured Grant Confirmation MAC CE
3. UE 在确认 MAC CE 传输后立即清除授权
```

#### DCI 格式（CS-RNTI）

DCI 格式 0_0/0_1 用于 CG Type 2 激活，携带：
- New data indicator（NDI）
- Modulation and coding scheme
- Frequency domain resource allocation
- Time domain resource assignment
- Frequency hopping flag

#### CG Type 2 与 Type 1 的对比

| 特性 | Type 1 | Type 2 |
|------|--------|--------|
| 配置方式 | 纯 RRC | RRC + PDCCH |
| 激活延迟 | 无（配置即生效） | 1~3 ms（PDCCH 周期） |
| 资源效率 | 始终占用资源 | 按需启用 |
| 灵活性 | 低（固定） | 高（可动态开关） |
| URLLC 适用性 | 周期性恒定 URLLC 流量 | 突发性 URLLC 流量 |

#### URLLC 应用场景

```
工厂自动化场景：
- URLLC 流量在生产周期内激活
- RRC 预配置 CG Type 2 periodicity = 5 ms
- gNB 在生产开始时 PDCCH 激活 → UE 开始传输
- gNB 在生产结束时 PDCCH 去激活 → UE 停止

优势：
- 非生产期间不占用资源
- 生产期间立即可用，无需 RRC 重新配置
```

---

### 3.3 Multiple CG Confirmation MAC CE（32 CG-fields）

#### 技术原理

**Multiple Entry Configured Grant Confirmation MAC CE** 是 Rel-16 引入的机制，允许 UE 在单个 MAC CE 中确认最多 32 个 CG 配置的状态（激活/去激活）。

> 📖 **规范来源**：TS 38.321 Clause 6.1.3.31

#### 消息结构

```
Octet 0: [CG0][CG1][CG2][CG3][CG4][CG5][CG6][CG7]
Octet 1: [CG8][CG9][CG10][CG11][CG12][CG13][CG14][CG15]
Octet 2: [CG16][CG17][CG18][CG19][CG20][CG21][CG22][CG23]
Octet 3: [CG24][CG25][CG26][CG27][CG28][CG29][CG30][CG31]

每个 CGi 字段：
  - 1 bit: CGi = 1 表示 PDCCH 收到了对 ConfiguredGrantConfigIndexMAC i 的激活/去激活
  - CGi = 0 表示没有收到
```

#### MAC Subheader

```
MAC Subheader:
  - R/R/E/LCID format
  - eLCID (extended LCID) for Multiple Entry CG Confirmation
  - Size: 2 octets (extended LCID)
```

#### UE 行为

```
当 CG Type 2 激活/去激活 PDCCH 被接收时：
1. MAC entity 触发 Configured Grant Confirmation
2. 如果 Multiple Entry CG Confirmation 已配置（或更高效）：
   - 生成 MAC CE，bitmap 指示哪些 CG 收到了 PDCCH
   - CGi = 1 if PDCCH received for CG index i
3. 传输 Confirmation MAC CE
4. 对于去激活的 CG：立即清除授权
5. 所有触发的 CG Confirmation 被取消
```

#### URLLC 应用场景

```
URLLC 场景（多组 CG 配置）：
- CG index 0: URLLC 控制信道，periodicity = 2 ms
- CG index 1: URLLC 数据信道，periodicity = 5 ms
- CG index 2: URLLC 数据信道，periodicity = 10 ms
- ...
- CG index 31: 最大 32 个 CG

当多个 CG 同时激活/去激活时：
- UE 发送单个 MAC CE（4 octets）确认所有状态
- 减少 MAC CE 开销
```

---

### 3.4 CG Retransmission Timer（自主重传）

#### 技术原理

**cg-RetransmissionTimer** 用于控制配置了 LBT（Listen-Before-Talk）的共享频谱信道上的自主重传行为。当 UE 在 CG 资源上 LBT 失败后，该 Timer 控制 UE 何时可以自主重传。

> 📖 **规范来源**：TS 38.321 Clause 5.8.2；Clause 5.21（LBT operation）

#### Timer 定义

```
cg-RetransmissionTimer:
  - Duration after a configured grant (re)transmission of a HARQ process
  - During this time, UE shall NOT autonomously retransmit
  - Timer starts at the beginning of the first symbol of PUSCH transmission

cg-SDT-RetransmissionTimer:
  - Duration after initial CG-SDT transmission with CCCH message
  - UE shall not autonomously retransmit
```

#### LBT 失败处理流程

```
LBT Failure Detection (per Serving Cell, per UL BWP):
1. MAC entity counts LBT failure indications
2. If LBT_COUNTER >= lbt-FailureInstanceMaxCount:
   → Consistent LBT failure triggered
   → Upper layers informed
   → Timer handling applies

On LBT failure indication:
1. HARQ process = pending
2. UE does NOT autonomously retransmit until cg-RetransmissionTimer expires
3. Network can schedule dynamic grant for retransmission
4. If cg-RetransmissionTimer expires and HARQ process still pending:
   → UE may autonomously retransmit using the same CG resource
```

#### Timer 行为详解

```
Timer Start/Restart Rules:
  - Timer STARTS at beginning of first symbol of PUSCH transmission
  - Timer RESTARTS on each transmission if LBT failure indication NOT received
  - Timer does NOT start/restart if LBT failure indication WAS received

Timeline Example:
  t=0: UE transmits PUSCH (LBT passed) → Timer starts
  t=4ms: LBT failure occurs → Timer NOT restarted
  t=8ms: Timer expires → UE can autonomously retransmit if pending

Alternative:
  t=0: UE transmits PUSCH (LBT passed) → Timer starts
  t=4ms: UE retransmits (LBT passed) → Timer restarts
  t=8ms: Timer expires → still no pending data → no action
```

#### URLLC 应用场景

```
工厂自动化（共享频谱如 5 GHz）：
- CG Type 2 配置 periodicity = 2 ms
- cg-RetransmissionTimer = 6 ms
- configuredGrantTimer running

Timeline:
  t=0: UE transmits on CG (LBT OK) → Timer starts
  t=2ms: Next CG transmission (LBT OK) → Timer restarts
  t=4ms: LBT failure → HARQ process pending, timer not restarted
  t=6ms: Timer expires → UE autonomously retransmits
  t=8ms: Network may also schedule dynamic grant

Benefit:
  - Autonomous retransmission without network scheduling delay
  - Timer prevents channel monopolization
  - Coexistence with other LBT devices
```

---

### 3.5 LCP Restrictions（逻辑信道映射限制）

#### 技术原理

**Logical Channel Prioritization (LCP) Restrictions** 是 MAC 层的重要机制，允许 RRC 限制逻辑信道只能使用特定的上行资源，从而确保 URLLC 流量使用最适合的资源。

> 📖 **规范来源**：TS 38.321 Clause 5.4.3.1（LCP procedure）

#### 限制参数表

| 参数 | 说明 | URLLC 用途 |
|------|------|-----------|
| **allowedSCS-List** | 允许的 SCS 列表 | URLLC 需要高 SCS（60kHz+） |
| **maxPUSCH-Duration** | 最大 PUSCH 持续时长 | URLLC 需要短传输（≤4 符号） |
| **configuredGrantType1Allowed** | 是否允许 Type 1 CG | URLLC 可绑定 grant-free 资源 |
| **allowedServingCells** | 允许的 Serving Cells | URLLC 可能只在特定小区 |
| **allowedCG-List** | 允许的 CG 配置列表 | URLLC 绑定特定 CG |
| **allowedPHY-PriorityIndex** | 允许的 PHY 优先级索引 | URLLC 高优先级 |
| **allowedHARQ-mode** | 允许的 UL HARQ 模式 | URLLC 可用特定 HARQ |

#### 逻辑信道选择算法

```
For each UL grant:
  For each logical channel j:
    IF ALL of the following are true:
      - allowedSCS-List includes UL grant SCS, AND
      - maxPUSCH-Duration >= PUSCH duration, AND
      - configuredGrantType1Allowed = true (if CG Type 1), AND
      - allowedServingCells includes cell, AND
      - allowedCG-List includes CG index, AND
      - allowedPHY-PriorityIndex includes priority, AND
      - allowedHARQ-mode includes HARQ mode
    THEN:
      logical channel j is eligible for this UL grant
```

#### 优先级处理顺序

UE MAC 按照以下优先级顺序处理上行数据（从高到低）：

```
1.  MAC CE for C-RNTI / UL-CCCH data
2.  MAC CE for (Enhanced) BFR / CG Confirmation / Multiple Entry CG Confirmation
3.  MAC CE for Sidelink CG Confirmation
4.  MAC CE for LBT failure
5.  MAC CE for Timing Advance Report
6.  MAC CE for SL-BSR
7.  MAC CE for (Extended) BSR（except padding）
8.  MAC CE for (Enhanced) Single/Multiple Entry PHR
9.  MAC CE for Positioning Measurement Gap Activation/Deactivation
10. MAC CE for Desired Guard Symbols
11. MAC CE for Case-6 Timing Request
12. MAC CE for (Extended) Pre-emptive BSR
13. Data from Logical Channels（except UL-CCCH）— 按 priority 排序
14. MAC CE for Recommended bit rate query
15. MAC CE for BSR（padding）
```

#### URLLC 配置示例

```RRC
LogicalChannelConfig ::= SEQUENCE {
  priority                       INTEGER (1..16),
  priorityLevelLCH               INTEGER (1..16),  -- 1 = highest
  allowedSCS-List                SEQUENCE OF SCS {
    scs-60kHz, scs-120kHz       -- 仅允许高 SCS
  }  OPTIONAL,
  maxPUSCH-Duration              ENUMERATED {
    to1, to2, to3, to4, to5, to6, to7, to8  -- symbols
  }  OPTIONAL,
  configuredGrantType1Allowed    ENUMERATED {true, false},
  allowedCG-List                 SEQUENCE OF INTEGER (0..7),
  allowedPHY-PriorityIndex        SEQUENCE OF INTEGER (1..16),
  ...
}
```

```
配置示例（URLLC 逻辑信道）：
  priority = 1 (highest)
  allowedSCS-List = {60kHz, 120kHz}
  maxPUSCH-Duration = to2 (2 symbols)
  configuredGrantType1Allowed = true
  allowedCG-List = {0}  -- 只使用 CG index 0

当 CG index 0 在 60kHz、2 符号 PUSCH 上发生时：
  → 只有 URLLC 逻辑信道满足所有限制
  → URLLC 数据立即传输
  → 与其他流量隔离
```

---

### 3.6 Enhanced Intra-UE Resource Prioritization

#### 技术原理

当多个上行授权在时间上重叠时，**Intra-UE Resource Prioritization**（`lch-basedPrioritization` 配置）控制 UE 如何决定优先级，确保高优先级流量（如 URLLC）优先传输。

> 📖 **规范来源**：TS 38.321 Clause 5.4.3.1（Resource allocation with lch-basedPrioritization）

#### 优先级判定算法

```
Step 1: 确定每个授权的优先级
  Grant Priority = 最高优先级的逻辑信道（该授权上有数据待传）

Step 2: 检测重叠授权
  对于每个时间上重叠的授权（PUSCH duration 重叠，在同一 BWP）：
    - 如果重叠授权优先级更高 → 本授权降优先级
    - 如果重叠授权优先级相等 → UE 实现决定
    - 如果重叠授权优先级更低 → 本授权优先

Step 3: 处理降优先级授权
  - configuredGrantTimer 停止（如果 autonomousTx PUSCH 已开始）
  - cg-RetransmissionTimer 停止（如果运行中）
  - 重叠的 SR 传输也被降优先级
```

#### Grant 类型默认优先级

```
1. RA grant（MAC RAR 或 fallbackRAR）：始终优先
2. CS-RNTI (NDI=1) / C-RNTI：检查是否有更高优先级 CG
3. Configured uplink grant：低于动态调度的高优先级信道
```

#### CI-RNTI 取消处理

如果 PUSCH 传输被 CI-RNTI（DCI 格式 1_0 或 0_1）取消：

```
→ CG 被降优先级
→ configuredGrantTimer 和 cg-RetransmissionTimer 停止
→ UE 等待新的授权
```

#### URLLC 应用场景

```
场景：URLLC 与 eMBB 流量同时到达
- Grant A（动态调度，C-RNTI）：URLLC 逻辑信道（priority=1）
- Grant B（Configured Grant）：eMBB 逻辑信道（priority=8）

两个 Grant 时间重叠：
→ Grant A 优先级更高（priority 数值越小优先级越高）
→ Grant A 被传输
→ Grant B 降优先级
→ 如果 Grant B 是 autonomousTx 且 PUSCH 已开始：
    configuredGrantTimer 停止
    cg-RetransmissionTimer 停止

Benefit: URLLC 流量保证在最佳资源上传输
```

---

### 3.7 Dedicated SR Configurations for URLLC

#### 技术原理

**Scheduling Request (SR)** 用于 UE 向 gNB 请求上行资源。URLLC 场景需要更频繁、更快速的 SR 机会，RRC 可以为 URLLC 逻辑信道配置专用的 SR 配置。

> 📖 **规范来源**：TS 38.321 Clause 5.4.4（Scheduling Request）；TS 38.331 `SchedulingRequestConfig`

#### 标准 SR 参数

```RRC
SchedulingRequestConfig ::= SEQUENCE {
  sr-ProhibitTimer          INTEGER (1..64)  OPTIONAL,  -- 单位：ms
  sr-TransMax               INTEGER (1..16)   OPTIONAL,  -- 最大 SR 传输次数
  ...
}
```

| 参数 | 说明 | 标准配置 | URLLC 配置建议 |
|------|------|-----------|---------------|
| sr-ProhibitTimer | SR 禁止定时器 | 较长（如 20ms） | 较短或为 0 |
| sr-TransMax | 最大 SR 尝试次数 | 8 | 4~8 |

#### URLLC SR 配置差异

| 特性 | 标准 SR | URLLC SR |
|------|---------|----------|
| sr-ProhibitTimer | 较长 | 较短或为 0 |
| SR 资源位置 | PCell | 可配置在 Scell |
| 每次 SR 机会 | 1 slot | 多个 slots |
| 用途 | 常规调度请求 | URLLC 快速资源请求 |

#### SR 触发场景（URLLC 特定）

SR 除了常规的数据触发外，还可以为以下场景触发：

```
1. SCell beam failure recovery
2. Consistent LBT failure recovery
3. Beam failure recovery of a BFD-RS set
4. Positioning measurement gap activation/deactivation
5. CG retransmission after LBT failure
```

#### SR 传输优先级

当 SR 与上行授权重叠时：

```
SR 传输优先于某些上行授权：
- RA grant 始终优先
- Configured grant（低优先级逻辑信道）可能被 SR 取代

SR 取消条件：
- UL grant 足够大，可以传输 SR 触发的数据
- 逻辑信道数据已被其他授权满足
```

---

### 3.8 BSR Enhancements

#### 技术原理

**Buffer Status Report (BSR)** 是 UE 向 gNB 报告上行缓冲区数据量的机制。URLLC 场景的 BSR 增强包括更快的触发和 Pre-emptive BSR。

> 📖 **规范来源**：TS 38.321 Clause 5.4.5（Buffer Status Reporting）

#### BSR 类型与触发条件

| BSR 类型 | 触发条件 | Format |
|----------|----------|--------|
| **Regular BSR** | 更高优先级的逻辑信道有数据到达 | Long/Short |
| **Periodic BSR** | periodicBSR-Timer 超时 | Long/Short |
| **Padding BSR** | 上行授权足够大，但有 padding | Truncated/Long |
| **Pre-emptive BSR** | IAB-MT 预占用父节点资源 | Extended |

#### URLLC 的 BSR 考虑

```
URLLC BSR 设计要点：
1. URLLC 逻辑信道应配置高优先级
2. Regular BSR 触发更快（数据到达立即触发）
3. 可配置较短的 periodicBSR-Timer
4. Padding BSR 优先传输（如果没有其他数据）
```

#### logicalChannelSR-DelayTimer

```RRC
LogicalChannelConfig ::= SEQUENCE {
  ...
  logicalChannelSR-DelayTimer    ENUMERATED {
    sf10, sf20, sf40, sf64, sf128, sf512, sf1024, sf2560, ...
  }  OPTIONAL,  -- BSR batching timer
  ...
}
```

```
logicalChannelSR-DelayTimer 的作用：
- URLLC 数据到达时，不立即触发 SR
- Timer 运行期间，数据累积
- Timer 超时后，触发单个 BSR 报告所有累积数据
- 减少 SR 开销，适合周期性 URLLC 流量
```

---

## 四、RLC 层增强

### 4.1 RLC UM Mode 替代 AM

#### 技术原理

**RLC UM（Unacknowledged Mode）** 是 URLLC 的首选 RLC 模式，因为它不进行 ARQ 重传，从而避免了因重传导致的延迟不确定性。

> 📖 **规范来源**：TS 38.322 Clause 5.1（RLC UM functional description）

#### AM vs UM 对比

| 特性 | AM Mode | UM Mode | URLLC 适用性 |
|------|---------|---------|-------------|
| 重传机制 | ARQ  retransmission | 无 | UM ✓ |
| Status Report | 有（POLL 触发） | 无 | UM ✓ |
| 处理延迟 | RTT + processing | 最低 | UM ✓ |
| 可靠性 | 高（保证交付） | 中等（无恢复） | 需要其他机制补偿 |
| 抖动 | 可变（取决于重传） | 可预测 | UM ✓ |
| 复杂度 | 高（状态机） | 低 | UM ✓ |

#### URLLC 选择 UM 的原因

```
URLLC 设计权衡：
- AM 的重传虽然提高可靠性，但引入了不可预测的延迟
- 对于工业控制回路，抖动比偶尔的丢失更不可接受
- 通过其他机制（Packet Duplication、Slot Aggregation）补偿可靠性
```

#### RLC UM 配置

```RRC
RLC-Config ::= SEQUENCE {
  am                         SEQUENCE { ... },
  um-BiDirectional           SEQUENCE {
    sn-FieldLengthUM         ENUMERATED {size6, size12},
    t-Reordering            ENUMERATED {ms0, ms1, ms2, ...}
  }  OPTIONAL,
  um-UniDirectionional       SEQUENCE {
    sn-FieldLengthUM         ENUMERATED {size6, size12},
    t-Reordering            ENUMERATED {ms0, ms1, ms2, ...}
  }  OPTIONAL,
  ...
}
```

```
URLLC 配置示例：
  RLC-Config = um-BiDirectional:
    sn-FieldLengthUM = size12  -- 12-bit SN，适合较长会话
    t-Reordering = ms2        -- 2ms 重组窗口
```

---

### 4.2 t-Reordering 配置

#### 技术原理

**t-Reordering** 是 RLC UM 接收端的定时器，用于处理 PDUs 的乱序到达和重组。

> 📖 **规范来源**：TS 38.322 Clause 5.3（t-Reordering timer specification）

#### t-Reordering 行为

```
接收端 RLC UM 实体行为：
1. 维护 RX_NEXT、RX_DELIV、RX_REORDER 等状态变量
2. 当 PDUs 乱序到达时，启动 t-Reordering
3. 如果缺失的 SN 在 Timer 超时前到达：
   → 交付完整序列到上层
4. 如果 Timer 超时仍未到达：
   → 交付当前可用的 PDUs 到上层
   → 缺失的 PDUs 被丢弃
```

#### t-Reordering 值与延迟

| t-Reordering 值 | 典型用途 | 附加延迟 |
|-----------------|----------|----------|
| **ms0** | 严格 URLLC | 0 ms |
| **ms1** | 极低延迟 URLLC | 1 ms |
| **ms2** | 低延迟 URLLC | 2 ms |
| **ms5** | URLLC（激进） | 5 ms |
| **ms10~ms20** | eMBB/Default | 10~20 ms |
| **ms35~ms100** | VoNR/Legacy | 35~100 ms |

#### SN 长度选择

| SN 长度 | 范围 | 适用场景 |
|---------|------|----------|
| 6 bits | 0~63 | 短会话、低延迟 |
| 12 bits | 0~4095 | 长会话、更高可靠性 |

```
SN wrap-around 处理：
RLC UM SN 到达上限后回绕。t-Reordering 用于处理乱序和回绕情况。
```

---

### 4.3 Polling 与 Status Report

#### 技术原理

RLC AM 使用 POLL 机制请求对端发送 Status Report，但 RLC UM 不支持 polling。

> 📖 **规范来源**：TS 38.322 Clause 5.4（Status reporting）

#### RLC AM Polling（与 URLLC 间接相关）

```
AM Polling 触发条件：
- 发送 polling bit = 1 的 PDU 后，轮询定时器运行
- 定时器超时或满足条件时，对端发送 Status Report

Status Report 包含：
- ACK_SN: 确认的最高 SN
- NACK_SN: 未确认的 SN 范围
- RWL_SN: AMD PDUs 窗口左边界
```

#### 与 PDCP 的交互

RLC 通过 PDCP 进行状态报告交互：

```
RLC UM 接收端：
- 不发送 Status Report 给发送端
- 但会通知 PDCP：
  - PDCP status report requested (via PDCP Control PDU)
  - PDCP t-Reordering 触发交付

RLC AM 接收端：
- 发送 Status Report（RX → TX）
- TX 根据 Status Report 触发重传
```

---

## 五、PDCP 层增强

### 5.1 Packet Duplication（数据包复制）

#### 技术原理

**Packet Duplication** 在 PDCP 层将同一个 PDCP PDU 复制多份，通过多个独立的 RLC 实体（和无线路径）传输，从而实现路径分集，提高可靠性。

> 📖 **规范来源**：TS 38.323 Clause 4.3.1（Duplication procedure）；Clause 5.1.5（PDCP duplication）

#### 架构

```
PDCP Data PDU
    ↓
    ├───────────────────→ Primary RLC Entity ──→ Cell Group 1 ──→ gNB-1
    │
    └───────────────────→ Secondary RLC Entity 1 ──→ Cell Group 2 ──→ gNB-2
    │
    └───────────────────→ Secondary RLC Entity 2 ──→ Cell Group 3 ──→ gNB-3 (optional)

PDCP 接收端：
- 从多个路径收到多份相同 PDCP PDU
- 交付第一份到达的到上层
- 丢弃后续重复副本
```

#### 激活/去激活方法

| 方法 | 说明 | 适用场景 |
|------|------|----------|
| **RRC 持久配置** | pdcp-Duplication = TRUE | 持续启用 duplication |
| **MAC CE 动态控制** | 1 octet: R | LCID | T | T=1 激活，T=0 去激活 | 按需启用/停用 |
| **Timer 超时自动去激活** |  inactivityTimer | 无数据传输后自动关闭 |
| **SCell 切换** | 复制路径随 sSCell 切换改变 | TDD 场景 |

#### MAC CE 格式

```
Octet: [R(1)][LCID(5)][T(1)][R(1)]

- R: Reserved (0)
- LCID: 逻辑信道 ID
- T: Duplication State (1 = ON, 0 = OFF)
```

#### RRC 配置

```RRC
PDCP-Config ::= SEQUENCE {
  drb                             SEQUENCE {
    pdcp-SN-Size                  ENUMERATED {size12, size18},
    discardTimer                  ENUMERATED {ms10, ms20, ms30, ...},
    pdcp-Duplication               BOOLEAN,  -- TRUE 启用 duplication
    t-Reordering                  ENUMERATED {ms0, ms1, ms2, ...},
    ...
  }  OPTIONAL,
  ...
}
```

#### 性能增益

```
无 Duplication：
  - BLER: ~10⁻³
  - P99 延迟: ~5-10 ms

有 Duplication（双路径）：
  - BLER: ~10⁻⁵ ~ 10⁻⁶（提升 100 倍）
  - P99 延迟: ~1-2 ms（改善 5 倍）

原理：两条独立路径同时失败的概率 ≈ P₁ × P₂（远小于单一路径）
```

---

### 5.2 PDCP t-Reordering

#### 技术原理

**PDCP t-Reordering** 是 PDCP 接收端的定时器，用于处理来自不同 RLC 实体（或因 HARQ 重传）导致的 PDCP PDU 乱序到达。

> 📖 **规范来源**：TS 38.323 Clause 5.1.2.1（Reception with t-Reordering）

#### 状态变量

| 变量 | 说明 |
|------|------|
| **RX_NEXT** | 下一个期望的 PDCP SN |
| **RX_DELIV** | 已交付到上层的最高 SN |
| **RX_REORDER** | t-Reordering timer 当前运行的 SN |

#### t-Reordering 行为

```
Step 1: PDCP PDU 到达（SN = X）
Step 2: 如果 X < RX_DELIV：
   → 丢弃（已交付过）
Step 3: 如果存在 SN 间隙：
   → 启动/重启 t-Reordering
Step 4: 如果 t-Reordering 超时：
   → 交付所有 SN <= RX_DELIV 的 PDUs
   → 更新 RX_DELIV
Step 5: 如果缺失的 SN 在 Timer 运行期间到达：
   → 交付连续序列
   → 停止/不重启 Timer
```

#### 配置值与延迟

| t-Reordering 值 | 用途 | 附加延迟 |
|-----------------|------|----------|
| **ms0** | 严格 URLLC | 0 ms |
| **ms1** | 极低延迟 | 1 ms |
| **ms2~ms5** | URLLC（激进） | 2~5 ms |
| **ms10~ms40** | eMBB | 10~40 ms |
| **ms160~ms200** | 覆盖扩展 | 160~200 ms |

---

### 5.3 PDCP 状态报告

#### 技术原理

PDCP 可以发送 **Status Report** 通知对端哪些 PDUs 已收到、哪些缺失。

> 📖 **规范来源**：TS 38.323 Clause 5.1.2.2（Status reporting）

#### Status Report 格式

```
PDCP Status Report:
- FMC (First Missing Count): 第一个缺失的 PDCP SN
- Bitmap: 后续 SN 的接收状态
- Bitmap size: 覆盖从 FMC 开始的 SN 范围
```

#### 触发条件

```
PDCP Status Report 触发：
1. t-Reordering 超时，存在未交付的 PDUs
2. 收到 PDCP Control PDU（请求 Status Report）
3. 应用于 AM 模式 DRB（RLC 重传导致）
```

---

## 六、HARQ 增强

### 6.1 紧凑型 HARQ-ACK 反馈

#### 技术原理

URLLC 需要快速 HARQ-ACK 反馈以降低延迟。通过紧凑的 UCI（Uplink Control Information）格式减少 ACK 开销。

> 📖 **规范来源**：TS 38.212 Clause 9.2.x（UCI on PUSCH）；TS 38.213 Clause 9.x（HARQ-ACK）

#### UCI 复用

```
PUSCH 上可以复用以下 UCI：
- HARQ-ACK（1~2 bits per TB）
- CSI (Channel State Information)
- PTRS (Phase Tracking Reference Signal) phase hint
- CG-UCI (Configured Grant UCI)

复用方式：
- HARQ-ACK + CSI: 按编码率分配资源
- HARQ-ACK + CG-UCI: HARQ-ACK 优先
```

#### Compact ACK 格式

```
HARQ-ACK 在 PUCCH 上传输：
- PUCCH format 0: 1~2 bits（极短反馈）
- PUCCH format 1: >2 bits（稍长反馈）
- ACK = 1, NACK = 0
```

### 6.2 PUCCH Cell Switching（sSCell）

#### 技术原理

**PUCCH Cell Switching** 允许 UE 在 TDD 系统中，将 PUCCH 传输切换到配置了上行资源的 sSCell（Secondary Cell in switching state），从而避免等待 PCell 的上行子帧。

> 📖 **规范来源**：TS 38.300 Clause 16.1.8；TS 38.331 `SCellConfig`

#### sSCell 配置

```RRC
SCellConfig ::= SEQUENCE {
  sCellIndex                  INTEGER (1..31),
  ...
  sCellConfigToAddMod         SEQUENCE {
    semiStaticConfig          SEQUENCE {
      -- 半静态时域 pattern
      subframePattern         BIT STRING,
      ...
    }  OPTIONAL,
    dynamicIndication         SEQUENCE {
      -- 动态切换指示
      dci-Format              ENUMERATED {format0_1, format1_1},
      ...
    }  OPTIONAL,
    ...
  }  OPTIONAL,
  ...
}
```

#### 切换控制方式

| 方式 | 说明 | 延迟降低 |
|------|------|----------|
| **半静态 Pattern** | RRC 配置周期性上行资源 | 中等 |
| **动态指示（DCI）** | PDCCH 动态指示切换小区 | 最大 |
| **MAC CE** | MAC CE 快速切换 | 较快 |

#### 工作流程

```
Without sSCell (Traditional TDD):
  PCell TDD: DL slot → UE 无法发送 PUCCH → 等待下一 UL slot
  等待时间: 0.5~1 ms (取决于 TDD 配置)

With sSCell Switching:
  PCell: DL slot → UE 切换到 sSCell（上行）→ 立即发送 PUCCH
  等待时间: ~0.01 ms（切换开销）
```

### 6.3 UCI on PUSCH 复用

#### UCI 类型与复用规则

```
UCI 复用优先级（从高到低）：
1. HARQ-ACK（最高）
2. CG-UCI（Configured Grant UCI）
3. CSI

复用规则：
- HARQ-ACK + CSI: HARQ-ACK 占用资源，CSI 编码率降低
- HARQ-ACK + CG-UCI: HARQ-ACK 优先，CG-UCI 在剩余资源上
```

### 6.4 CBG-based Retransmission

#### 技术原理

**CBG（Code Block Group）Retransmission** 允许只重传出错的 Code Block Group，而不是整个 Transport Block。

> 📖 **规范来源**：TS 38.212 Clause 9.x（CBG retransmission）

#### CBG 配置

```RRC
PDSCH-Config ::= SEQUENCE {
  ...
  codeBlockGroupTransmission    SEQUENCE {
    maxCodeBlockGroupsPerTB     INTEGER (2..8),
    ...
  }  OPTIONAL,
  ...
}
```

```
maxCodeBlockGroupsPerTB:
  - 2: 1 TB 分 2 个 CBG
  - 8: 1 TB 分 8 个 CBG
```

#### DCI 字段

```
DCI 1_1 中的 CBG 字段：
- CBGTI (Code Block Group Transmission Information): 指示本次传输的 CBGs
- CBGFL (Code Block Group Flush): 刷新 CBG 窗口
```

---

## 七、RRC 层配置

### 7.1 BWP 配置与 URLLC

#### URLLC BWP 设计原则

```
URLLC BWP 配置要点：
1. 独立 SCS: 60kHz 或 120kHz（针对 URLLC 优化）
2. 较小带宽: 减少 UE 处理复杂度
3. 专用调度资源池: 与 eMBB 隔离
4. 独立 CORESET: URLLC PDCCH 专用搜索空间
```

#### BWP 配置示例

```RRC
-- URLLC BWP 配置示例（符合 TS 38.331 BWP-Downlink IE 语法）
BWP-Downlink ::= SEQUENCE {
  locationAndBandwidth         1000,   -- 50MHz 带宽对应值（~270 PRBs @ 60kHz）
  subcarrierSpacing           kHz60,  -- 60kHz SCS，适合 URLLC 低延迟
  cyclicPrefix                normal
}
```

### 7.2 SR 配置（更频繁调度请求）

#### URLLC SR 配置

```RRC
SchedulingRequestConfig ::= SEQUENCE {
  schedulingRequestToAddModList  SEQUENCE OF SchedulingRequest {
    sr-ResourceId               INTEGER (0..7),
    sr-ConfigId                 INTEGER (0..7),
    reportConfigId              INTEGER (0..7),  -- 关联 reportConfig
    ...
  },
  ...
}

SchedulingRequest ::= SEQUENCE {
  sr-ResourceId               INTEGER (0..7),
  sr-ProhibitTimer            INTEGER (1..64)  OPTIONAL,  -- ms
  sr-TransMax                 INTEGER (1..16)  OPTIONAL,
  ...
}
```

```
URLLC SR 配置（与标准配置对比）：
  sr-ProhibitTimer = 0 (无禁止期，最大频率)
  sr-TransMax = 4 (快速失败判定)

优势：
  - 更快的 SR 响应
  - 更短的上行资源请求延迟
```

### 7.3 Logical Channel SR 配置

#### 逻辑信道与 SR 绑定

```RRC
LogicalChannelConfig ::= SEQUENCE {
  logicalChannelGroup          INTEGER (0..7)   OPTIONAL,
  logicalChannelSR-Config      SEQUENCE {
    sr-ProhibitTimer           INTEGER (1..64)  OPTIONAL,
    sr-TransMax                INTEGER (1..16)  OPTIONAL,
    ...
  }  OPTIONAL,
  ...
}
```

```
配置 URLLC 逻辑信道 SR：
  logicalChannelSR-Config:
    sr-ProhibitTimer = 0
    sr-TransMax = 4

每个逻辑信道可以有不同的 SR 配置：
  - URLLC: 最短禁止期、最高优先级
  - eMBB: 标准配置
```

### 7.4 UE Capability 报告

#### URLLC UE Capability

```RRC
UE-NR-Capability ::= SEQUENCE {
  ...
  urllc-Mode                   SEQUENCE {
    mini-Slot-BasedScheduling  BOOLEAN,
    slot-Aggregation           BOOLEAN,
    processingCapability1      BOOLEAN,
    ...
  }  OPTIONAL,
  ...
  featureSet                  SEQUENCE OF FeatureSet,
  featureSetCombinations       SEQUENCE OF FeatureSetCombination,
  ...
}
```

```
URLLC UE 能力上报项：
1. mini-Slot-BasedScheduling: 支持 mini-slot 调度
2. slot-Aggregation: 支持 slot aggregation
3. processingCapability1: 支持 Processing Capability I（8 symbols）
4. dmrs-Bundling: 支持 DMRS bundling
5. multipleCG: 支持多个 CG 配置
```

---

## 八、高层多连接（Multi-Connectivity）

### 8.1 双 PDU Session 架构

#### 技术原理

URLLC 的多连接通过建立两个独立的 PDU Session 实现，两个 Session 通过不相交的路径连接到 5GC，提供端到端的路径冗余。

> 📖 **规范来源**：TS 23.501 Clause 5.6.8（Multi-connectivity for URLLC）；TS 38.300 Clause 16.1.6

#### 架构图

```
                        ┌──────────────────────────────────────┐
                        │            5G Core (5GC)             │
                        │                                      │
                        │  ┌──────────┐      ┌──────────┐     │
                        │  │  SMF 1   │      │  SMF 2   │     │
                        │  │(Primary) │      │ (Secondary)│   │
                        │  └─────┬────┘      └─────┬────┘     │
                        │        │ N4                 │ N4      │
                        │  ┌─────┴────┐      ┌─────┴────┐     │
                        │  │  UPF 1   │      │  UPF 2   │     │
                        │  │(Primary) │      │(Secondary)│     │
                        │  └─────┬────┘      └─────┬────┘     │
                        │        N3                 N3         │
                        └────────┼───────────────────┼───────────┘
                                 │                   │
                        ┌────────┴───────────────────┴────────┐
                        │              NG-RAN Layer              │
                        │                                       │
                        │  ┌──────────────────┐ ┌─────────────┐ │
                        │  │      gNB-1       │ │    gNB-2    │ │
                        │  │    (Primary)     │ │  (Secondary)│ │
                        │  └────────┬─────────┘ └──────┬──────┘ │
                        │           │                   │        │
                        └───────────┼───────────────────┼────────┘
                                     │                   │
                               ┌─────┴───────────────────┴─────┐
                               │              UE                 │
                               │   PDU Session 1 (Primary)      │
                               │   PDU Session 2 (Secondary)     │
                               └────────────────────────────────┘
```

#### 路径隔离要求

```
NG-RAN 隔离要求：
  - 两个 PDU Session 的 DRB 必须连接到不同的 NG-RAN nodes
  - 或者即使在同一 NG-RAN node，两个 NG-U tunnel 必须通过不相交的传输层路径
  - 如果 NG-RAN 无法满足不相交要求 → 向 5GC 报告 failure

UL 复制：
  - UE 发送相同数据到两个路径
  - 两个路径独立接收

DL 去重：
  - 每个 UPF 独立转发到对应的 gNB
  - gNB/UE 检测并丢弃重复数据包
```

### 8.2 同一 UPF 的冗余传输

#### 技术原理

即使在同一 UPF 的情况下，也可以建立两个 NG-U tunnels，通过不同的传输层路径提供冗余。

> 📖 **规范来源**：TS 38.300 Clause 16.1.6.2

#### 架构

```
                        ┌────────────────────────────────┐
                        │            5GC (SMF)            │
                        │                                 │
                        │  ┌──────────────────────────┐   │
                        │  │          UPF             │   │
                        │  │   (replication point)    │   │
                        │  └──────────┬───────┬───────┘   │
                        │             │ N3    │ N3         │
                        │    ┌───────┘       └───────┐    │
                        │    │                         │    │
                        │  Path 1                   Path 2  │
                        │ (disjoint transport layer paths)  │
                        └────────────────────────────────┘
```

#### 5GC 指示机制

```
每个 QoS Flow 有一个 redundant transmission indicator：
  - 5GC 向 NG-RAN 指示哪些 QoS Flow 需要冗余传输
  - NG-RAN 根据指示配置 DRB 的冗余传输

DL 去重：
  - NG-RAN 接收来自两个 tunnels 的相同数据包
  - NG-RAN 消除重复数据包（per QoS Flow）

UL 复制：
  - NG-RAN 复制数据包并通过两个 tunnels 发送
  - 两个 UPF 分别接收
```

### 8.3 ATSSS（流量 Steering/Splitting）

#### 技术原理

**ATSSS（Access Traffic Steering, Switching, Splitting）** 是 5GC 提供的机制，允许在多个接入路径之间进行流量引导、切换和分裂。

> 📖 **规范来源**：TS 23.501 Clause 5.6.8A（ATSSS）

#### ATSSS 架构

```
UE 侧：
  ┌─────────────┐
  │ ATSSS Client│ ←── 5GC 配置 ATSSS rules
  └──────┬──────┘
         │
    ┌────┴────┐
    │PDU Session│
    │(multi-access)│
    └────┬────┘
         │
    ┌────┴────────────────┐
    │                     │
  5G Access           WLAN Access
  (NR)                (Wi-Fi)

5GC 侧：
  - SMF 控制 ATSSS rules
  - ATSSS rules 指示如何分配流量到不同接入
  - 可配置为：active-standby、load-balancing、priority-based
```

#### ATSSS Rules 类型

| 类型 | 说明 | URLLC 适用性 |
|------|------|-------------|
| **Active-Standby** | 主路径传输，备用路径待命 | ✓（高可靠） |
| **Load-Balancing** | 流量分配到多个路径 | ✗（延迟不确定） |
| **Priority-based** | 按优先级选择路径 | ✓（适合 URLLC） |

---

## 九、Unlicensed 频段 URLLC

### 9.1 LBT/Channel Access 机制

#### 技术原理

在共享频谱（如 5 GHz Unlicensed）上，URLLC 使用 **LBT（Listen-Before-Talk）** 机制来与其他设备共享信道。

> 📖 **规范来源**：TS 38.300 Clause 16.1.7；TS 37.213（LBT operation for shared spectrum）

#### Channel Access 模式

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **Semi-static channel occupancy** | gNB/UE 发起信道占用，持续时间预先定义 | 周期性 URLLC |
| **Dynamic channel access** | gNB 基于实时感知决定传输 | 动态 URLLC 流量 |
| **Enhanced dynamic channel access** | 高速率传输场景 | 突发 URLLC |

#### LBT 过程

```
LBT 步骤：
1. Energy Detection: 检测信道是否空闲
2. if 信道空闲 for CCA Duration:
   → 传输可以开始
3. if 信道忙:
   → 执行 extended CCA（更长的检测窗口）
   → 成功后开始传输

CCA Duration: 9 μs（常规）
Extended CCA: 可变，取决于竞争程度
```

### 9.2 CG Retransmission Timer 配置

#### 配置参数

```RRC
ConfiguredGrantConfig ::= SEQUENCE {
  ...
  cg-RetransmissionTimer      INTEGER (1..64)   OPTIONAL,
  ...
}
```

```
cg-RetransmissionTimer 值：
  - 1: 最短 timer，最快自主重传
  - 64: 最长 timer，最保守

Timer 选择考虑：
  - 较短: 高可靠性要求，低延迟
  - 较长: 避免与与其他 LBT 设备冲突
```

### 9.3 Enhanced Intra-UE Overlapping Resource Prioritization

#### 技术原理

当配置了 `lch-basedPrioritization` 且 CG Retransmission Timer 时，增强的优先级机制控制多个重叠资源的选择。

> 📖 **规范来源**：TS 38.321 Clause 5.4.3.1

#### 优先级决策

```
Enhanced Intra-UE Resource Prioritization:
1. Grant Priority = highest priority logical channel data pending
2. For each overlapping grant:
   - if overlapping grant has higher priority LCH:
     → this grant de-prioritized
   - else:
     → this grant prioritized
3. De-prioritized grants:
   - configuredGrantTimer stopped
   - cg-RetransmissionTimer stopped
   - Overlapping SR transmissions also de-prioritized
```

---

## 十、时间敏感通信（TSC）

### 10.1 TSC 架构

#### 技术原理

**TSC（Time Sensitive Communications）** 是 URLLC 在工业 IoT 场景的扩展，强调确定性通信和/或等时通信，具有高可靠性和可用性。

> 📖 **规范来源**：TS 38.300 Clause 16.8；TS 23.501 Clause 5.6.6

#### TSC 特性

```
TSC 定义（TS 23.501）：
  - Deterministic communication: 时间确定的通信
  - Isochronous communication: 等时通信（周期性）
  - High reliability and availability: 高可靠性和可用性

典型用例（TS 22.104）：
  - 工业自动化
  - cyber-physical control applications
  - 运动控制（motion control）
```

### 10.2 5G 系统时间参考（10ns 粒度）

#### 时间同步机制

```
5G 系统时间参考：
  - 粒度: 10 ns
  - 由 gNB 通过 RRC 信令发送给 UE
  - 发送方式: 单播或广播

用途:
  - UE 与网络的时间同步
  - TSC 场景下的时延补偿
  - 多 UE 协调
```

#### RRC 配置

```RRC
SystemInformationBlockType31 ::= SEQUENCE {
  ...
  tsc-TimeReferenceInfo        SEQUENCE {
    timeReferenceSFN           INTEGER (0..1023),
    timeDomainOffset           INTEGER (0..399),
    ...
  }  OPTIONAL,
  ...
}
```

### 10.3 TSN Bridge 集成

#### TSN（Time Sensitive Networking）集成

```
5G 系统作为 TSN Bridge：
  ┌─────────────────────────────────────────────────────┐
  │               5G System (as TSN Bridge)             │
  │                                                     │
  │  TSN Time ══╪═╗  5G Time ══╪═╗                     │
  │            ╔═╝  ╚══════╗  ╔═╝                      │
  │            ║          ║  ║                          │
  │          Ingress    Egress                          │
  │          Port        Port                           │
  └─────────────────────────────────────────────────────┘

UE 作为 TSN Bridge 端口：
  - 5G 系统提供时间同步到 UE
  - UE 将 5G 时间同步到本地 TSN 网络
  - 实现端到端的确定性通信
```

---

## 十一、Inter-UE Prioritization

#### 技术原理

**Inter-UE Prioritization** 主要指 sidelink（PC5）场景下的 UE 间协调机制，用于 V2X 和 URLLC 场景。

> 📖 **规范来源**：TS 38.321 Clause 5.22（Sidelink Inter-UE Coordination）

#### Sidelink Inter-UE Coordination 流程

```
Step 1: UE-A 触发 Sidelink Inter-UE Coordination Request
Step 2: UE-A 通过 PC5-RRC 发送协调请求到 UE-B
Step 3: UE-B 评估资源状态，发送协调信息（可用资源、冲突等）
Step 4: UE-A 根据协调信息选择最佳传输资源
Step 5: 传输在协商的资源上进行

Benefits:
  - 减少 sidelink 冲突
  - 提高 URLLC V2X 可靠性
  - 协调资源分配
```

#### 与空口 URLLC 的区别

```
空口 URLLC Inter-UE Prioritization:
  - 通过 LCP Restrictions 和 Intra-UE Prioritization 实现
  - 同一 UE 内的逻辑信道优先级

Sidelink URLLC:
  - 通过 Sidelink Inter-UE Coordination 实现
  - 不同 UE 之间的协调
```

---

## 十二、R15/R16 功能总结

### R15 特性

| 特性 | 规范 | 核心机制 |
|------|------|----------|
| **Mini-slot 调度** | TS 38.214 Clause 6.1.2/7.1 | 1~14 符号传输，Type B mapping |
| **Type 1 CG** | TS 38.321 Clause 5.8.2 | RRC 配置，无 PDCCH 激活 |
| **Type 2 CG** | TS 38.321 Clause 5.8.2 | RRC + PDCCH 激活 |
| **Slot Aggregation** | TS 38.214 Clause 6.1.3 | TB repetition，最大 32 slots |
| **LDPC BG2** | TS 38.212 Clause 5.2.2 | 低码率编码，3 dB 增益 |
| **LCP Restrictions** | TS 38.321 Clause 5.4.3.1 | 逻辑信道资源映射限制 |
| **Packet Duplication** | TS 38.323 Clause 5.1.5 | PDCP 复制到多 RLC entity |
| **CQI Table 3（10⁻⁵ BLER）** | TS 38.214 Clause 5.2.2 | URLLC 可靠性 CQI 表 |
| **DCI Format 1_1/0_1** | TS 38.212 Clause 7.3 | CBG retransmission |
| **PUCCH Cell Switching** | TS 38.300 Clause 16.1.8 | sSCell 切换（TDD） |
| **Multi-Connectivity (intra)** | TS 38.300 Clause 16.1.6 | PDCP duplication |

### R16 特性

| 特性 | 规范 | 核心机制 |
|------|------|----------|
| **CG Retransmission Timer** | TS 38.321 Clause 5.8.2 | LBT 失败后自主重传 |
| **Multiple Entry CG Confirmation** | TS 38.321 Clause 6.1.3.31 | 32 CG-fields 确认（Octet bitmap） |
| **Enhanced Intra-UE Prioritization** | TS 38.321 Clause 5.4.3.1 | lch-basedPrioritization 算法 |
| **CG-SDT** | TS 38.321 Clause 5.27 | CG-based Small Data Transmission |
| **Multiple SPS** | TS 38.331 sps-Config | 多个 SPS 配置并发 |
| **DMRS Bundling** | TS 38.214 Clause 6.1.3 | 跨 slots DMRS 连续估计 |
| **N3 Interface Redundancy** | TS 23.501 Clause 5.6.8 | 双 UPF 冗余路径 |
| **ATSSS** | TS 23.501 Clause 5.6.8A | 多接入流量控制 |

---

## 十三、Rel-17 URLLC 增强

> Rel-17（3GPP Release 17，2022 年 6 月完成）进一步增强了 URLLC 能力，主要包括：紧凑型 DCI 调度（DDCI 0_2）、增强的可靠性机制（Enhanced Reliability）、以及多 TCI 状态控制信道增强。

---

### 13.1 DCI 0_2：紧凑型 UL 调度（Compact DCI for URLLC）

#### 13.1.1 设计背景

DCI 0_1 是通用的 UL 调度格式，包含 80+ 比特字段。对于 URLLC 场景（如工厂自动化中高频度的控制信号），这些额外开销会造成：

- PDCCH 盲检复杂度高（UE 需要检测大量候选 PDCCH）
- 编码开销大（DCI 越大，编码所需CCE越多）
- 调度延迟增加（紧凑场景下 DCI 解析时间也重要）

Rel-17 引入 DCI 0_2（Compact UL Grant），专门针对短包、频繁调度的 URLLC 场景优化。

#### 13.1.2 DCI 0_2 字段（与 DCI 0_1 对比）

| 字段 | DCI 0_1 比特数 | DCI 0_2 比特数 | 变化 |
|------|-------------|-------------|------|
| Identifiers (DCI format) | 1+1 | 1+1 | 相同 |
| Frequency domain resource | ~18 | ~15 | 减少（更多预配置） |
| Time domain assignment | 4 | 3 | 减少（更小表格） |
| Frequency hopping flag | 1 | 0 | 删除（由 RRC 配置） |
| MCS | 5 | 4 | 减少（MCS 表缩小） |
| Redundancy version | 2 | 1 | 减少（RV 序列简化） |
| HARQ process number | 4 | 3 | 减少（最多 8 HARQ 进程） |
| Pointer to frequency domain | 0 | 1 | 新增（指示预配置资源） |
| **Total** | **~80 bits** | **~30 bits** | **↓62%** |

> 📖 **规范来源**：TS 38.212 Clause 7.3.1.1.2（DCI format 0_2）和 TS 38.214 Clause 7.1.1（UL resource allocation for DCI 0_2）

#### 13.1.3 DCI 0_2 使用场景

```
DCI 0_2 适用场景：

  - URLLC 上行业务（mini-slot 调度，1~2 symbols）
  - 高频度周期性传输（如工业控制信号 1 ms 周期）
  - 预配置资源池（frequency domain 由 higher layer 配置）
  - 固定 MCS 或有限 MCS 选择（由 RRC 指定表格）

DCI 0_2 不适合场景：
  - 动态频率选择性调度（需要完整 frequency domain allocation）
  - 大带宽传输（需要大 MCS 范围）
  - HARQ 进程数 > 8（需要 DCI 0_1）
```

#### 13.1.4 DCI 0_2 的 frequency domain allocation

DCI 0_2 中 frequency domain resource 字段减少的原因：frequency domain allocation 可以由 RRC 预配置。

```
预配置模式：

RRC 配置（ConfiguredGrantConfig）：
  └─ frequencyDomainAlloc = 预定义的 RB 集合（由 CG 配置指定）
  └─ frequency Hopping 由 cg-Configured（由 RRC 配置，非 DCI 指示）

DCI 0_2 中的 frequency domain 字段：
  - 如果配置了 frequencyDomainAlloc：
      → DCI 只指示 pointer/offset（1~2 bits）
  - 如果未配置：
      → fallback 到完整的 frequency domain allocation（约 15 bits）

优势：减少 DCI 开销，提高 URLLC 调度效率
```

---

### 13.2 Enhanced Reliability（增强可靠性）

#### 13.2.1 增强内容概述

Rel-17 在 Rel-15/16 基础上进一步增强 URLLC 可靠性：

| 特性 | Rel-15/16 | Rel-17 增强 |
|------|-----------|------------|
| **LDPC BG2** | 仅 DL | DL + UL（上行也支持 BG2 for short TB） |
| **DMRS Bundling** | Intra-slot bundling | Inter-slot bundling（更长时间窗口） |
| **Multiple TX** | Packet Duplication（2~3 paths） | 多达 4 个 RLC entities |
| **PDCCH 可靠性** | 1 slot aggregation | 2~4 slots PDCCH 监听增强 |

#### 13.2.2 UL LDPC Base Graph 2 扩展

Rel-15/16 中 LDPC BG2 仅用于 DL。Rel-17 将 BG2 扩展到 UL，使上行短传输块也能享受 BG2 的 3 dB 编码增益：

```
BG2 上行扩展条件（同时满足）：
  - TBS <= 256 bits（或由网络配置）
  - MCS <= 4（QPSK ~ 16QAM）
  - UE 支持 urllc-UL-Enhanced-Reliability
  - 网络配置 enabledULBGr2 = true
```

配置 IE（TS 38.331）：
```RRC
ConfiguredGrantConfig ::= SEQUENCE {
  ...
  enhancedReliability          ENUMERATED {enabled}  OPTIONAL,   -- Rel-17，启用 UL BG2
  ...
}
```

#### 13.2.3 Inter-slot DMRS Bundling

Rel-15/16 DMRS Bundling 只支持 intra-slot（同一 slot 内的多个符号）。Rel-17 将 bundling 扩展到 inter-slot（跨多个 slots）：

```
Inter-slot DMRS Bundling 机制：

RRC 配置（BWP-Downlink）：
  dmrs-Bundling ::= SEQUENCE {
    bundlingSize         ENUMERATED {n2, n4, n6, n8},   -- 跨 slots 数
    bundlingInterval     ENUMERATED {slotLevel, subFrameLevel},
    ...
  }

工作模式（bundlingSize = n4）：
  - Slot 0: PDSCH + DMRS symbol 0
  - Slot 1: PDSCH（无 DMRS），UE 使用 slot 0 的 DMRS 估计
  - Slot 2: PDSCH（无 DMRS），UE 使用 slot 0 的 DMRS 估计
  - Slot 3: PDSCH + DMRS symbol 0
  - UE 对这 4 个 slots 进行联合信道估计

优势：
  - 增加等效 DMRS 资源（时间分集增益）
  - 提高 SNR（平均噪声降低）
  - 适合 slot aggregation（4~8 slots）
  - 可靠性提升约 2~3 dB
```

> 📖 **规范来源**：TS 38.214 Clause 6.2.2（PDSCH DMRS configuration）和 Clause 6.1.3（DMRS bundling for URLLC）

#### 13.2.4 多达 4 个 RLC Entity 的 Packet Duplication

Rel-15/16 的 PDCP Duplication 支持最多 3 个路径（1 primary + 2 secondary）。Rel-17 扩展到最多 4 个路径（1 primary + 3 secondary）：

```
RLC Entity 扩展配置（RRC）：

PDCP-Config ::= SEQUENCE {
  ...
  ul-DataCompression    SEQUENCE { supported BOOLEAN }  OPTIONAL,
  pdcp-Duplication      SEQUENCE {
    duplInfoList        SEQUENCE (SIZE(1..3)) OF
                        SEQUENCE {
      sdap-Config       SDAP-Config,
      rlc-Stack         SEQUENCE (SIZE(2..4)) OF RLC-Config,
                        -- 最多 4 个 RLC entities
    }  OPTIONAL,
    ...
  }  OPTIONAL,
  ...
}

注意：
  - 3 个 secondary RLC entities = 4 个 total 路径
  - 每个路径有独立的 Cell Group / 载波
  - 激活/去激活由 MAC CE 控制（T-bit per LCID）
  - 与 multi-connectivity 结合，实现极致可靠性
```

#### 13.2.5 PDCCH 监听增强（PDCCH Monitoring Robustness）

Rel-17 为 URLLC 控制信道增强了可靠性：

```
PDCCH 可靠性增强：

1. 多 PDCCH 候选配置：
   - UE 可配置多个 CORESETs，每个 CORESET 对应不同 reliability 级别
   - URLLC 控制信道使用更低的 aggregation level（AL 2/4，而非 AL 8/16）

2. 跨 CORESET 的盲检：
   - UE 在多个 CORESET 上监听同一 DCI
   - 任一 CORESET 检测成功即触发传输

3. PDCCH 监听周期调整：
   - URLLC BWP 配置更短的 monitoring period（mini-slot 级别）
   - network-controlled fast PDCCH switching

4. DCI 格式选择：
   - DCI 1_0（最简单）用于高可靠性场景
   - DCI 1_1 用于需要更多字段的场景
   - DCI 0_2（Rel-17）用于紧凑 URLLC UL 调度

配置示例（RRC）：
  ControlResourceSet ::= SEQUENCE {
    frequencyDomainResources   BIT STRING,
    duration                    INTEGER (1..3),   -- 1~3 symbols
    tcid-State                  SEQUENCE {
      tcid                      INTEGER (0..15),
      ...
    }  OPTIONAL,
    ...
  }
```

---

### 13.3 Multiple TCI States for URLLC Control Channels

#### 13.3.1 多 TCI 状态背景

TCI（Transmission Configuration Indicator）用于指示 UE 的波束/准 co-location 关系。Rel-15/16 中，PDCCH 和 PDSCH 各支持单个 TCI state。Rel-17 扩展到多个 TCI states，适用于 URLLC 场景：

#### 13.3.2 Multiple TCI States for PDCCH

```
Rel-17 PDCCH TCI 配置：

ControlResourceSet ::= SEQUENCE {
  ...
  tci-PresentInDCI         ENUMERATED {enabled, disabled},
  tci-States               SEQUENCE (SIZE(1..maxNrofTCI-States)) OF
                           TCI-State,
  tci-Pattern              SEQUENCE {
    tci-PatternPeriod       INTEGER (1..10),
    tci-Offset              INTEGER (0..9),
    ...
  }  OPTIONAL,
  ...
}

使用场景：
  - UE 快速切换 beam（不同 TCI states 对应不同 beam directions）
  - 适用于高铁/车载场景（快速信道变化）
  - URLLC 控制信道可靠性提升

DCI 指示逻辑：
  - tci-PresentInDCI = enabled：
      → DCI 中有 TCI field（1~3 bits），指示当前使用的 TCI state
  - tci-PresentInDCI = disabled：
      → UE 使用预配置的单一 TCI state（无 DCI 指示）

切换时间：
  - TCI state switch delay = 1 symbol（switchingTime）
  - 与 sSCell PUCCH switching 机制类似
```

#### 13.3.3 Multiple Active TCI States for PDSCH

```
Rel-17 PDSCH 多 TCI 状态：

PDSCH-Config ::= SEQUENCE {
  ...
  tci-StatesToAddModList    SEQUENCE (SIZE(1..8)) OF TCI-State,
  tci-Pattern               SEQUENCE {
    tci-PatternPeriod        INTEGER (1..10),
    ...
  }  OPTIONAL,
  ...
}

多 TCI 使用场景：
  - Spatial division multiplexing（空间分隔）
  - 同一 UE 在不同 beam 上接收多个 PDSCH 层
  - 提高频谱效率（URLLC + 高吞吐联合）

限制：
  - 多 TCI states 适用于 multi-TRP 场景
  - 需要 UE 支持 multipleActiveConfig-IndexingPerServingCell
  - 网络侧需要多个 TRP（Transmission Reception Points）
```

---

### 13.4 repKARQ-Procedure：重复 HARQ 过程

#### 13.4.1 repKARQ-Procedure 机制

repKARQ-Procedure（Repetition based HARQ Procedure）是 CG Type 2 的可选配置，用于 URLLC 场景下的无 HARQ-ACK 传输：

```
repKARQ-Procedure 配置：

ConfiguredGrantConfig ::= SEQUENCE {
  ...
  configuredGrantType2       SEQUENCE {
    periodicity              ENUMERATED {n2, n5, n10, ...},
    nrofHARQ-Processes       INTEGER (1..16),
    repKARQ-Procedure        BOOLEAN,   -- TRUE = 启用重复 HARQ
    ...
  }  OPTIONAL,
  ...
}

当 repKARQ-Procedure = TRUE 时：

  1. 同一个 TB 的多个重复传输使用不同的 RV 序列
       - Repetition 1: RV = 0
       - Repetition 2: RV = 2
       - Repetition 3: RV = 3
       - Repetition 4: RV = 1
       → 4 次传输后完成一个完整 redundancy package

  2. UE 不需要发送 HARQ-ACK：
       - gNB 假设所有 repetitions 都被接收
       - 无 PUCCH/PUSCH HARQ-ACK 反馈（节省开销）
       - 适合极低延迟场景（不允许 ACK/NACK 反馈延迟）

  3. CG 配置的 nrofHARQ-Processes 同时指示：
       - HARQ process pool 大小
       - 最大并发 repetition 数

配置示例：
  CG Type 2 配置：
    - periodicity = n2（2 ms）
    - nrofHARQ-Processes = 8
    - repKARQ-Procedure = TRUE
    - nrofConfiguredGrantTimes = 4

  传输模式：
    - 每个 CG 时机传输一个 repetition
    - 4 次连续 CG 时机完成一个 TB 的完整传输
    - 无需 HARQ-ACK，网络自动组合 4 个 repetition
```

#### 13.4.2 repKARQ vs 标准 HARQ 对比

| 维度 | 标准 HARQ | repKARQ-Procedure |
|------|-----------|-------------------|
| HARQ-ACK | 必须反馈 | 无反馈 |
| 延迟 | 1 RTT（K1 symbols）| 0（RTT 延迟消除）|
| 可靠性 | 靠 ARQ 纠正错误 | 靠 redundancy 避免错误 |
| 频谱效率 | 较高（有 ACK 机制）| 较低（重复传输）|
| 适用场景 | 普通 URLLC | 极低延迟 + 高可靠性 |

> 📖 **规范来源**：TS 38.321 Clause 5.8.2（Configured Grant Type 2 with repKARQ-Procedure）和 TS 38.214 Clause 7.1.2（UL transmission with configured grant repetition）

---

### 13.5 configuredGrantType1Allowed 与 configuredGrantType2Allowed 的共存关系

#### 13.5.1 概念区分

同一个 Logical Channel 可以配置同时使用 CG Type 1 和 CG Type 2，由 `configuredGrantType1Allowed` 和 `configuredGrantType2Allowed` 两个独立参数控制：

```
LogicalChannelConfig ::= SEQUENCE {
  ...
  ul-SpecificParameters          SEQUENCE {
    priority                     INTEGER (1..16),
    allowedSCS-List              ENUMERATED {scs-15kHz, scs-30kHz, ...},
    maxPUSCH-Duration            ENUMERATED {ms0p5, ms0p5-1, ...},
    allowedCodingType           ENUMERATED {fullBuffer, partial, ...},  -- Rel-17
    configuredGrantType1Allowed  BOOLEAN,   -- CG Type 1 许可
    configuredGrantType2Allowed  BOOLEAN,   -- CG Type 2 许可
    ...
  }  OPTIONAL,
  ...
}

配置组合：

  Case A: configuredGrantType1Allowed = TRUE, configuredGrantType2Allowed = FALSE
    → 该 LCH 只能使用 CG Type 1
    → 优点：无激活延迟，配置后立即生效

  Case B: configuredGrantType1Allowed = FALSE, configuredGrantType2Allowed = TRUE
    → 该 LCH 只能使用 CG Type 2
    → 优点：网络可动态激活/去激活，资源利用更灵活

  Case C: configuredGrantType1Allowed = TRUE, configuredGrantType2Allowed = TRUE
    → 该 LCH 可同时使用 Type 1 和 Type 2
    → LCP 限制决定优先级（见下方）

  Case D: configuredGrantType1Allowed = FALSE, configuredGrantType2Allowed = FALSE
    → 该 LCH 不能使用任何 CG（只能 dynamic grant）
```

#### 13.5.2 LCP Restrictions 中的优先级处理

当两个 CG 类型都被允许时，MAC 实体需要根据 LCP Restrictions 决定使用哪个：

```
LCP Restrictions 决策逻辑：

1. LCP Restrictions 检查顺序：
   Step 1: 检查 LCH 的 allowedSCS-List
   Step 2: 检查 maxPUSCH-Duration
   Step 3: 检查 configuredGrantType1Allowed / configuredGrantType2Allowed

2. 配置示例：
   LogicalChannelConfig for URLLC-LCH:
     - configuredGrantType1Allowed = TRUE
     - configuredGrantType2Allowed = TRUE

   此时 MAC 实体的选择逻辑：
   ┌─────────────────────────────────────────────────────┐
   │ Grant Type 1（CG Type 1）优先级 > Grant Type 2（CG Type 2）│
   │                                                      │
   │ 如果 CG Type 1 可用：                                │
   │   → 使用 CG Type 1（立即传输，无激活延迟）           │
   │                                                      │
   │ 如果 CG Type 1 不可用（如 configuredGrantTimer 超时）：│
   │   → 降级到 CG Type 2（需要 PDCCH 激活）              │
   │                                                      │
   │ 如果 CG Type 2 也不可用：                           │
   │   → 触发 SR，请求动态调度                           │
   └─────────────────────────────────────────────────────┘

3. 配置建议：
   - 高优先级 URLLC LCH：同时允许 Type 1 + Type 2
     → 确保最佳延迟（Type 1）和最大灵活性（Type 2）
   - 中优先级 URLLC LCH：只允许 Type 2
     → 网络可动态管理资源分配
   - 低优先级 LCH：不允许 CG，使用 dynamic grant
     → 保证 URLLC 资源预留
```

---

### 13.6 allowedPHY-PriorityIndex 的使用场景

#### 13.6.1 PHY Priority Index 概念（勘误）

> ⚠️ **重要修正**：早期版本描述"P0~P3 四级优先级"——这是不准确的。根据 TS 38.321 Clause 5.4.3.1，`allowedPHY-PriorityIndex` 只有两个有效值：
> - `0` = lowPriority（对应普通 eMBB/LCP 优先级）
> - `1` = highPriority（对应 URLLC 高优先级资源）
>
> 该参数用于区分同一 UE 中不同逻辑信道的资源优先级，高优先级信道可使用 dedicated 的 PHY 资源池。

```RRC
-- 符合 TS 38.321 的 LCP Restrictions 配置
LogicalChannelConfig ::= SEQUENCE {
  ...
  ul-SpecificParameters       SEQUENCE {
    priority                  INTEGER (1..16),
    allowedPHY-PriorityIndex SEQUENCE (SIZE(1..2)) OF
                              INTEGER (0..1),  -- 0=lowPriority, 1=highPriority
    ...
  }  OPTIONAL,
  ...
}
```

**配置示例：**

| 信道类型 | allowedPHY-PriorityIndex | 说明 |
|---------|--------------------------|------|
| URLLC 控制信道 | `{1}` | 只能使用 highPriority 资源 |
| URLLC 数据信道 | `{1}` 或 `{0,1}` | 可用高优先级，也可在低负载时用普通资源 |
| eMBB 信道 | `{0}` | 只能使用 lowPriority 资源 |

#### 13.6.2 使用场景

```
allowedPHY-PriorityIndex 典型应用：

场景 1：工厂自动化（多优先级 URLLC）

  传感器类型 A（紧急停止信号）：
    allowedPHY-PriorityIndex = {1} (highPriority)
    → 只能使用高优先级 PHY 资源池
    → 可靠性最高，延迟最低

  传感器类型 B（状态监控数据）：
    allowedPHY-PriorityIndex = {0, 1} (lowPriority + highPriority)
    → 高负载时用 highPriority，低负载时可用 lowPriority
    → 平衡延迟和资源利用率

场景 2：V2X（动态优先级切换）

  V2X Uu 接口（蜂窝链路）：
    allowedPHY-PriorityIndex = {1} (highPriority)
    → 始终使用高优先级资源

  V2X PC5 接口（直通链路）：
    allowedPHY-PriorityIndex = {1} (highPriority)
    → PC5 链路始终最高优先级

场景 3：触觉互联网（多模态数据）

  触觉反馈数据（0.5 ms 延迟要求）：
    allowedPHY-PriorityIndex = {1} (highPriority)
    → 最高优先级

  音频数据（10 ms 延迟要求）：
    allowedPHY-PriorityIndex = {0} (lowPriority)
    → 普通优先级
```

#### 13.6.3 与 LCP Restrictions 其他参数的关系

```
LCP Restrictions 参数优先级顺序（从高到低）：

1. priority（逻辑信道优先级）：决定信道间调度顺序
2. allowedPHY-PriorityIndex：决定资源块优先级
3. allowedSCS-List：决定时域资源可用性
4. maxPUSCH-Duration：决定单次传输持续时间上限
5. configuredGrantType1/2Allowed：决定 CG 类型可用性

调度决策流程：
  Step 1: 按 priority 从高到低处理 LCH
  Step 2: 对每个 LCH，检查 allowedPHY-PriorityIndex 是否匹配当前可用资源
  Step 3: 检查 allowedSCS-List、maxPUSCH-Duration 等
  Step 4: 选择匹配的 CG 或 dynamic grant
```

---

### 13.7 dmrs-Bundling 在不同场景下的配置要求

#### 13.7.1 dmrs-Bundling 场景分类

dmrs-Bundling 根据使用场景分为三种配置模式：

```
dmrs-Bundling 配置模式：

模式 1：Intra-slot Bundling（同一 slot 内多个 DMRS 符号）
  - 适用：2~4 个符号的 mini-slot PDSCH
  - 配置：BundlingInterval = intraSlot（默认）
  - 效果：相邻 DMRS 符号进行联合信道估计

模式 2：Inter-slot Bundling（跨多个 slots）
  - 适用：Slot Aggregation（4~8 slots）
  - 配置：BundlingSize = n4 或 n8
  - 效果：多个 slots 的 DMRS 进行时间分集估计

模式 3：Dynamic Bundling（根据信道条件动态选择）
  - 适用：信道快速变化场景（如车载）
  - 配置：bundlingInterval = dynamicSelection
  - 效果：UE 根据 CQI 反馈自动选择 bundling 模式
```

#### 13.7.2 各场景详细配置

```
场景 1：工厂自动化（静态信道，低移动性）

RRC 配置（BWP-Downlink）：
  pdsch-Config ::= SEQUENCE {
    dmrs-Bundling ::= SEQUENCE {
      bundlingSize         n4,           -- 4 slots 聚合
      bundlingInterval     slotLevel,
      ...
    },
    ...
  }

  预期效果：
    - DMRS bundling across 4 slots
    - 信道估计增益：~2 dB
    - 可靠性提升：BLER 从 10⁻³ → 10⁻⁵

场景 2：V2X（高移动性，快速信道变化）

RRC 配置（BWP-Downlink）：
  pdsch-Config ::= SEQUENCE {
    dmrs-Bundling ::= SEQUENCE {
      bundlingSize         n2,           -- 2 slots（更短窗口）
      bundlingInterval     slotLevel,
      additionalDMRS-DCI   enabled,       -- 额外 DMRS via DCI
      ...
    },
    ...
  }

  预期效果：
    - 更短的 bundling 窗口适应快速变化
    - additionalDMRS-DCI 在 DCI 1_1 中指示额外 DMRS 位置
    - 平衡可靠性与跟踪速度

场景 3：URLLC eMBB 混合部署（资源隔离）

RRC 配置（BWP-Downlink）：
  pdsch-Config ::= SEQUENCE {
    dmrs-Bundling ::= SEQUENCE {
      bundlingSize         n4,
      bundlingInterval     slotLevel,
      bundlingForDCI-Format1-0-enabled  TRUE,   -- DCI 1_0 也支持 bundling
      ...
    },
    ...
  }

  预期效果：
    - URLLC PDSCH 使用 bundling（DCI 1_1）
    - eMBB PDSCH 使用标准 DMRS（无 bundling）
    - 资源隔离，避免 URLLC bundling 影响 eMBB 调度
```

#### 13.7.3 DMRS Bundling 与 PDCCH 监听冲突处理

当 dmrs-Bundling 与 PDCCH 监听冲突时（如 UE 需要盲检 PDCCH，无法同时处理 DMRS bundling），配置原则：

```
冲突处理规则：

1. PDSCH processing 与 PDCCH monitoring 并行：
   - UE 在 PDSCH 处理期间继续 PDCCH 监听
   - dmrs-Bundling 跨 PDCCH monitoring symbols 进行
   - 不冲突，因为 PDCCH 和 PDSCH 使用不同DMRS资源

2. Bundling 窗口内的 PDCCH 监听：
   - 如果 PDCCH scheduling 跨越 bundling 窗口：
     → UE 优先处理 PDCCH decoding
     → 暂停 bundling estimate（不完全停止）
   - 如果 PDCCH 检测成功：
     → 触发新 PDSCH reception
     → bundling 从新 PDSCH 重新开始

3. 配置建议：
   - 在 URLLC BWP 上启用 dmrs-Bundling
   - 在相同 BWP 上配置较少的 PDCCH candidates（减少冲突）
   - 或将 PDCCH 配置在另一个 CORESET（不同 BWP）
```

---

### 13.8 Rel-17 URLLC 特性总结表

| 特性 | 规范章节 | 适用方向 | 关键增益 |
|------|---------|---------|---------|
| **DCI 0_2** | TS 38.212 Clause 7.3.1.1.2 | UL 调度 | DCI 开销减少 62%，适合高频调度 |
| **UL LDPC BG2** | TS 38.214 Clause 5.2.2 | UL 可靠传输 | ~3 dB 编码增益，适合短 TB |
| **Inter-slot DMRS Bundling** | TS 38.214 Clause 6.1.3 | Slot Aggregation | ~2~3 dB SNR 提升 |
| **4-path Packet Duplication** | TS 38.323 Clause 4.3.1 | 极高可靠性 | 可靠性提升 10× |
| **Multiple TCI (PDCCH)** | TS 38.214 Clause 5.1.2 | 快速 beam switch | 切换延迟 1 symbol |
| **Multiple TCI (PDSCH)** | TS 38.214 Clause 5.1.3 | Multi-TRP SDM | 频谱效率提升 |
| **repKARQ-Procedure** | TS 38.321 Clause 5.8.2 | 无 ACK 传输 | 消除 HARQ-ACK 延迟 |
| **Enhanced Reliability IE** | TS 38.331 Clause 5.6.1 | UE 能力上报 | 网络据此配置 URLLC 特性 |

---

## 十四、端到端流程与交互序列

> 本章节以流程图和序列图的形式，展示 URLLC 各层机制如何串联工作，涵盖传输路径、处理时序、配置交互和数据路由四个维度。

---

### 14.1 端到端传输路径总览

URLLC 场景下有三条典型传输路径，分别对应不同的可靠性/延迟需求：

#### 路径 A：Grant-Free（CG Type 1，无 PDCCH 激活）

```
UE(MAC)                              gNB(MAC)                    gNB(PHY)
  │                                    │                           │
  │  [1] CG 配置已存储                  │                           │
  │  ←─────────────────────────────  RRC Reconfiguration          │
  │                                    │                           │
  │  [2] 数据到达 MAC（URLLC 数据包）    │                           │
  │  ──────────────────────────────────►                          │
  │  [3] 查找匹配的 CG 配置             │                           │
  │  (LCP Restrictions 过滤)          │                           │
  │  [4] PUSCH 准备（零调度延迟）        │                           │
  │  ───────────────────────────────────────►  PUSCH transmission  │
  │                                    │      (pre-configured RB)  │
  │                                    │  [5] PDCCH (CS-RNTI) NACK?│
  │                                    │  [6] CG Retransmission?   │
  │                                    │                           │
  │  [7] HARQ-ACK (PUCCH/PUSCH)        │  [8] HARQ result          │
  │  ───────────────────────────────────────►  PUSCH + UCI        │
  │                                    │                           │
  ▼                                    ▼                           ▼
```

**关键特征**：数据到达 → PUSCH 传输 = 0 RTT，无调度等待。

#### 路径 B：动态调度（含 PDCCH 授权）

```
UE                                    gNB                               Time
 │                                     │                                │
 │  [1] 数据到达 MAC                   │                                │
 │  ──────────────────────────────────► SR trigger                     │
 │                                     │                                │
 │  [2] SR on PUCCH (SR资源配置)        │                                │
 │  ──────────────────────────────────►                                 │
 │                                     │  [3] PDCCH (C-RNTI)            │
 │                                     │  DCI 0_0/0_1 ← UL grant        │
 │                                     │◄────────────────────────────────
 │  [4] PUSCH preparation              │                                │
 │  (N2 symbols processing)           │                                │
 │  ───────────────────────────────────────► PUSCH transmission         │
 │                                     │                                │
 │  [5] HARQ-ACK                      │                                │
 │  (K1 >= N1+1 symbols)              │                                │
 │  ───────────────────────────────────────► UCI on PUCCH/PUSCH       │
 │                                     │                                │
 ▼                                     ▼                                ▼
```

**关键特征**：数据到达 → PUSCH 传输 = 1~3 slots（含 SR + PDCCH 等待）。

#### 路径 C：Packet Duplication + Slot Aggregation（最高可靠性）

```
UE(PDCP)                             UE(MAC)                          gNB
 │                                      │                              │
 │  [1] PDCP Data PDU 到达             │                              │
 │  ─► Duplicate → Primary RLC Ent.    │                              │
 │  ─► Duplicate → Secondary RLC Ent.  │                              │
 │                                      │                              │
 │  Primary Path:                      │  Secondary Path:             │
 │  [2a] PUSCH preparation             │  [2b] PUSCH preparation       │
 │  [3a] CG/Grant PUSCH #1             │  [3b] CG/Grant PUSCH #2       │
 │  ───────────────────────────────────────────────────────────────► │
 │  (slot aggregation 1 of N)         │  (slot aggregation 1 of N)    │
 │                                      │                              │
 │  Primary: HARQ-ACK #1               │  Secondary: HARQ-ACK #2      │
 │  ───────────────────────────────────────────────────────────────► │
 │                                      │                              │
 │  [4] PDCP 接收双路径数据              │                              │
 │  - 第一份到达 → 交付上层             │                              │
 │  - 副本 → 丢弃                       │                              │
 │  [5] PDCP t-Reordering timer 处理   │                              │
 │  [6] 完成重组，交付高层              │                              │
 ▼                                      ▼                              ▼
```

**关键特征**：双路径并行传输，任一路径成功即可交付，可靠性提升 100 倍。

---

### 14.2 Grant-Free 完整传输流程

本节以时间顺序展示 CG Type 1 的端到端处理步骤。

#### Step 0：RRC 配置阶段

```
UE ←═══════════════════════════════════════════ gNB
     RRCReconfiguration
     └─ ConfiguredGrantConfig (Type 1)
        ├─ cs-RNTI = 0x0001
        ├─ timeDomainOffset = 0
        ├─ periodicity = n5 (5ms)
        ├─ frequencyDomainAlloc = 0101...0 (RB bitmap)
        ├─ mcs-Table = qam64LowSE (URLLC)
        └─ nrofHARQ-Processes = 8

     ← Acknowledge
UE MAC entity stores configured uplink grant
```

#### Step 1：周期性传输时机计算

```
当 SFN=0, slot=0, symbol=0 时：

传输时机 = timeDomainOffset + N × periodicity
         = 0 + N × 5ms

N = floor[(SFN × 1024 × 14 + slot × 14 + symbol) / periodicity]

HARQ Process ID = floor[N mod 8]
```

#### Step 2：数据到达触发 MAC PDU 组装

```
UE MAC:
  1. 数据包到达 Logical Channel (priority=1, URLLC)
  2. LCP 检查：allowedSCS-List ⊇ 60kHz ✓
  3. LCP 检查：maxPUSCH-Duration >= 4 symbols ✓
  4. LCP 检查：configuredGrantType1Allowed = true ✓
  5. 数据 → MAC SDU 组装
  6. MAC subheader 添加 (LCID = URLLC_LCH)
  7. MAC CE（如有 BSR/PHR）组装
  8. MAC PDU → RLC
  9. RLC UM segment → PDU (SN assigned)
  10. PDCP duplicate → 两个 RLC entities
  11. PUSCH transmission on configured grant
```

#### Step 3：HARQ-ACK 接收与 Timer 处理

```
gNB 接收 PUSCH → 解码成功：
  → HARQ-ACK = 1 (ACK)
  → PDCP ← RLC ← MAC ← PHY

gNB 接收 PUSCH → 解码失败：
  → HARQ-ACK = 0 (NACK)
  → gNB 可选择：
     a) 通过 CS-RNTI 调度 dynamic retransmission
     b) 如果配置了 cg-RetransmissionTimer：
        UE 在 timer 超时后自动重传
```

---

### 14.3 动态调度完整传输流程

#### 流程概览

```
UE                                                      gNB
 │                                                        │
 │  ┌─────────────────────────────────────────────────┐  │
 │  │ Step 1: SR 触发                                  │  │
 │  │  - URLLC 数据到达                                 │  │
 │  │  - BSR 触发（logicalChannelSR-DelayTimer 超时）  │  │
 │  │  - SR on PUCCH (configured SR resource)           │  │
 │  └─────────────────────────────────────────────────┘  │
 │  ──────────────────────────────────────────────────►  │
 │                                                        │
 │  ┌─────────────────────────────────────────────────┐  │
 │  │ Step 2: PDCCH 调度                                │  │
 │  │  - DCI 0_1 (C-RNTI)                               │  │
 │  │  - UL grant: MCS + RB allocation + K2 offset      │  │
 │  │  - K2 = 1 slot (Type A) 或 K2 = 0 (Type B)       │  │
 │  └─────────────────────────────────────────────────┘  │
 │  ◄──────────────────────────────────────────────────  │
 │                                                        │
 │  ┌─────────────────────────────────────────────────┐  │
 │  │ Step 3: PUSCH 准备                                │  │
 │  │  - N2 symbols processing time (Capability I)   │  │
 │  │  - 15kHz: N2 = 8 symbols = 533 μs               │  │
 │  │  - 60kHz: N2 = 3 symbols = 50 μs                │  │
 │  └─────────────────────────────────────────────────┘  │
 │  ──────────────────────────────────────────────────►  │
 │                    PUSCH transmission                   │
 │                                                        │
 │  ┌─────────────────────────────────────────────────┐  │
 │  │ Step 4: HARQ-ACK 反馈                             │  │
 │  │  - K1 >= N1 + 1 symbols                           │  │
 │  │  - UCI on PUCCH format 0/1 或 UCI on PUSCH      │  │
 │  └─────────────────────────────────────────────────┘  │
 │  ──────────────────────────────────────────────────►  │
 │                                                        │
 ▼                                                        ▼
```

#### Mini-slot 动态调度时序（Type B）

```
Slot N (30kHz SCS, 0.5ms/slot):
  Symbol 0-1:   DCI 0_1 received (PDCCH)
  Symbol 2-5:   K2 = 1 symbol (PUSCH starts symbol 2)
  Symbol 2-3:   PUSCH transmission (2-symbol mini-slot)
  Symbol 4-8:   UE processing (N2 = 5 symbols @ 30kHz)
  Symbol 9:     UE ready for next transmission

端到端延迟（不含回程）：
  PUSCH transmission: 67 μs × 2 = 133 μs
  UE processing: 5 symbols × 33.33 μs = 167 μs
  Total: ~300 μs
```

---

### 14.4 UE PHY 处理时序图

#### PDSCH → HARQ-ACK 完整时序

以下时序图展示 UE 接收 PDSCH 后，到发送 HARQ-ACK 的完整处理路径，包括 Capability I/II 的具体数值。

```
PDSCH Processing Timeline (Capability I)

Legend:
  DCI_rx  = DCI received (PDCCH)
  PDSCH   = PDSCH reception
  Proc    = UE processing (decode + ACK preparation)
  K1      = PDSCH to HARQ-ACK offset
  UCI_tx  = HARQ-ACK transmission


15 kHz SCS (symbol = 66.67 μs, slot = 1 ms):
┌──────────────────────────────────────────────────────────────────────┐
│ DCI_rx   PDSCH      Proc                    K1>=N1+1   UCI_tx        │
│   │        │         │                         │          │           │
│   ▼        ▼         ▼                         ▼          ▼           │
│ [S0]    [S0]    [S0..S7]                   [S9..S13]  [S9..S13]     │
│   │        │         │                         │          │           │
│   K0=0     K0=0      N1=8 symbols             K1=9 sym   (same slot) │
│            ───────────────── 1 ms slot ──────────────────           │
└──────────────────────────────────────────────────────────────────────┘
  RTT = 1 slot (PUCCH in same slot as DCI) = 1 ms

30 kHz SCS (symbol = 33.33 μs, slot = 0.5 ms):
┌──────────────────────────────────────────────────────────────────────┐
│ DCI_rx   PDSCH      Proc                K1>=N1+1   UCI_tx          │
│   │        │         │                     │          │             │
│   ▼        ▼         ▼                     ▼          ▼             │
│ [S0]    [S0]    [S0..S4]               [S5..S9]  [S5..S9]       │
│   │        │         │                     │          │             │
│   K0=0     K0=0      N1=5 symbols         K1=6 sym   (same slot)   │
│            ───────────────── 0.5 ms slot ──────────────────        │
└──────────────────────────────────────────────────────────────────────┘
  RTT = 0.5 ms

60 kHz SCS (symbol = 16.67 μs, slot = 0.25 ms):
┌──────────────────────────────────────────────────────────────────────┐
│ DCI_rx   PDSCH   Proc      K1>=N1+1   UCI_tx                         │
│   │        │       │          │          │                            │
│   ▼        ▼       ▼          ▼          ▼                            │
│ [S0]    [S0]  [S0..S2]   [S3..S6]   [S3..S6]                        │
│   │        │       │          │          │                            │
│   K0=0     K0=0    N1=3 sym   K1=4 sym  (same slot)                  │
│            ───────────────── 0.25 ms slot ──────────────────         │
└──────────────────────────────────────────────────────────────────────┘
  RTT = 0.25 ms

120 kHz SCS (symbol = 8.33 μs, slot = 0.125 ms):
┌──────────────────────────────────────────────────────────────────────┐
│ DCI_rx  PDSCH  Proc     K1>=N1+1   UCI_tx                            │
│   │       │      │        │         │                               │
│   ▼       ▼      ▼        ▼         ▼                               │
│ [S0]   [S0] [S0..S1]  [S2..S3]  [S2..S3]                           │
│   │       │      │        │         │                               │
│   K0=0    K0=0   N1=2 sym  K1=3 sym  (same slot)                    │
│           ───────────────── 0.125 ms slot ──────────────────        │
└──────────────────────────────────────────────────────────────────────┘
  RTT = 0.125 ms
```

#### Capability I vs Capability II 数值表

| SCS | N1 (Capability I) | N2 (Capability I) | N1 (Capability II) | N2 (Capability II) |
|-----|------------------|------------------|-------------------|-------------------|
| 15 kHz | 8 sym = 533 μs | 8 sym = 533 μs | 13 sym = 867 μs | 13 sym = 867 μs |
| 30 kHz | 5 sym = 167 μs | 5 sym = 167 μs | 10 sym = 333 μs | 10 sym = 333 μs |
| 60 kHz | 3 sym = 50 μs | 3 sym = 50 μs | 7 sym = 117 μs | 7 sym = 117 μs |
| 120 kHz | 2 sym = 17 μs | 2 sym = 17 μs | 5 sym = 42 μs | 5 sym = 42 μs |

> 📖 **规范来源**：TS 38.214 Clause 6.2（PDSCH processing timeline）和 Clause 7（PUSCH processing timeline）

#### PUSCH Preparation → Transmission Timeline

```
PUSCH preparation timeline (UE side, dynamic grant):

收到 DCI (symbol S_DCI):
  │
  ├─ K2 = offset from DCI to PUSCH start
  │
  ▼
PUSCH starts at symbol S_PUSCH = S_DCI + K2:
  │
  ├─ RLC UM: segment SDU into PDUs
  ├─ PDCP: add PDCP header, duplicate if configured
  ├─ MAC: assemble MAC PDU
  │
  ▼
UE 必须在前 N2 symbols 内完成所有处理

Timeline (60kHz, Capability I, K2=0):
  Symbol 0:    DCI 0_1 received
  Symbol 0-2:  UE processing runs in parallel with PDSCH reception (N2=3 symbols, 50 μs total)
  Symbol 3:    PUSCH transmission starts (K2=0 means PUSCH starts same slot as DCI)
  ─────────────────────────────────────
  Total preparation: 3 × 16.67 μs = 50 μs (done in parallel, no added latency)

Timeline (60kHz, Capability I, K2=1 slot):
  Symbol 0:    DCI 0_1 received
  Symbol 0:    K2 = 1 slot → PUSCH scheduled at next slot
  Symbol 0-2:  UE processing (N2=3 symbols, completes well before next slot)
  Symbol 14+:  Next slot begins, PUSCH transmission starts
  ─────────────────────────────────────
  UE has 0.25 ms - 50 μs = 200 μs slack time before PUSCH
```

---

### 14.5 CG Type1/Type2 配置与激活完整流程

#### 14.5.1 ConfiguredGrantConfig RRC IE 结构

```RRC
ConfiguredGrantConfig ::= SEQUENCE {
  -- 标识与激活
  configuredGrantTimer           INTEGER (1..64)          OPTIONAL,
  frequencyDomainAlloc           BIT STRING (SIZE(18)),    -- PUSCH 频域资源
  cs-RNTI                        INTEGER (0..65535),       -- CS-RNTI 地址
  cg-RetransmissionTimer        INTEGER (1..64)          OPTIONAL,
  frequencyHopping              ENUMERATED {
    interSlot, intraSlot
  }                                           OPTIONAL,

  -- 时域配置（Type 2 用，Type 1 通过 RRC 直接设置）
  timeDomainAlloc               INTEGER (0..15)           OPTIONAL,
  mcs-Table                     ENUMERATED {
    qam64, qam256, qam64LowSE
  }                                          OPTIONAL,
  mcs-TableTransformPrecoder   ENUMERATED {...}          OPTIONAL,
  transformPrecoder            ENUMERATED {
    enabled, disabled
  }                                          OPTIONAL,

  -- CG-SDT 相关
  cg-DTX-PreambleUnderlay      SEQUENCE {...}           OPTIONAL,

  -- 允许多个 CG（Rel-16）
  nrofConfiguredGrantTimes     INTEGER (1..64)          OPTIONAL,

  -- Type 2 激活参数（Type 1 不使用）
  configuredGrantType2          SEQUENCE {
    periodicity              ENUMERATED {...},
    nrofHARQ-Processes      INTEGER (1..16),
    repKARQ-Procedure       BOOLEAN,
    ...
  }                                          OPTIONAL,

  -- CG index（用于 Multiple Entry CG Confirmation）
  configuredGrantConfigIndex   INTEGER (0..7)           OPTIONAL,
  ...
}
```

#### 14.5.2 Type 1 CG 配置流程（RRC-Only，无 PDCCH）

```
RRC 配置流程：

gNB                                    UE
 │                                      │
 │  RRCReconfiguration                 │
 │  ├─ spCellConfig                     │
 │  │   └─ initialUplinkBWP              │
 │  │       └─ configuredGrantConfig    │
 │  │           ├─ cs-RNTI = 0x0001      │
 │  │           ├─ periodicity = n2     │
 │  │           ├─ timeDomainOffset = 0 │
 │  │           ├─ frequencyDomainAlloc  │
 │  │           │   = 0101...0101        │
 │  │           ├─ mcs-Table = qam64LowSE│
 │  │           └─ [no configuredGrantType2]  ← Type 1 标识
 │  │                                      │
 │  ├─ LogicalChannelConfig              │
 │  │   ├─ priority = 1                   │
 │  │   ├─ allowedSCS-List = {scs-60kHz} │
 │  │   ├─ maxPUSCH-Duration = to4       │
 │  │   └─ configuredGrantType1Allowed   │
 │  │       = true                        │
 │  │                                      │
 │  ├─ mac-MainConfig                     │
 │  │   └─ ul-GrantConfig                 │
 │  │       └─ configuredGrantConfigToAddModList
 │  │                                      │
 │  └─ rrcReconfigurationComplete        │
 │  ◄───────────────────────────────────│
 │                                      │
 │  UE 应用配置：                        │
 │  MAC entity stores configured uplink grant
 │  计算周期性传输时机                   │
 │  CG Type 1 即刻生效，无激活步骤       │
 ▼                                      ▼
```

#### 14.5.3 Type 2 CG 配置与激活流程（RRC + PDCCH）

```
Phase 1: RRC 预配置（Base Configuration）

gNB                                    UE
 │                                      │
 │  RRCReconfiguration                 │
 │  ├─ configuredGrantConfig            │
 │  │   ├─ configuredGrantType2         │  ← Type 2 标识
 │  │   │   ├─ periodicity = n5        │
 │  │   │   ├─ nrofHARQ-Processes = 8  │
 │  │   │   └─ repKARQ-Procedure = true │
 │  │   ├─ cs-RNTI = 0x0002            │
 │  │   └─ [无 timeDomainOffset]        │  ← 激活时才设置
 │  │                                      │
 │  └─ RRCReconfigurationComplete       │
 │  ◄───────────────────────────────────│
 │                                      │
 │  UE MAC: 存储 base config            │
 │  CG state: INACTIVE (not yet valid)  │
 ▼                                      ▼

Phase 2: PDCCH 激活

gNB                                    UE
 │                                      │
 │  PDCCH (DCI 0_0/0_1, CS-RNTI)        │
 │  ├─ Time domain assignment (K2)      │
 │  ├─ Frequency domain alloc          │
 │  ├─ MCS                             │
 │  ├─ NDI = 0 (new transmission)      │
 │  └─ New data indicator             │
 │  ◄──────────────────────────────────│
 │                                      │
 │  UE MAC:                            │
 │  1. 解析 DCI，确认 CG Type 2 激活     │
 │  2. 存储上行授权（基于 DCI + base config）
 │  3. 启动 configuredGrantTimer        │
 │  4. 触发 Configured Grant Confirmation MAC CE
 │  5. CG state: ACTIVE               │
 │                                      │
 │  UE 在周期性资源上传输               │
 ▼                                      ▼

Phase 3: PDCCH 去激活

gNB                                    UE
 │                                      │
 │  PDCCH (DCI 0_0/0_1, CS-RNTI)        │
 │  ├─ NDI = 1 (deactivation)          │
 │  │  （或特定去激活字段）              │
 │  ◄──────────────────────────────────│
 │                                      │
 │  UE MAC:                            │
 │  1. 触发 Multiple CG Confirmation   │
 │  2. 传输 Confirmation MAC CE       │
 │  3. 立即清除 configured grant      │
 │  4. CG state: INACTIVE             │
 ▼                                      ▼
```

#### 14.5.4 激活/去激活 PDCCH DCI 字段说明

```
DCI 0_1 for CG Type 2 activation/deactivation:

| Field                      | Bits | Description                          |
|---------------------------|------|--------------------------------------|
| Identifiers               | 1+1  | Reserved, NDI (0=new, 1=retrans)   |
| Time domain assignment    | 4    | K2 offset (Type A/B)                 |
| Frequency domain alloc    | ~18  | PUSCH RB allocation                 |
| Frequency hopping flag    | 1    | Inter/intra slot hopping             |
| MCS                        | 5    | MCS index                          |
| Redundancy version        | 2    | RV sequence                         |
| HARQ process number      | 4    | HARQ process ID                     |
| CBGTI (if CBG enabled)   | 4-8  | Code Block Group info               |
| CGI (Multiple CG)        | 5    | CG index (for confirmation)         |

CG Type 2 activation 的判断逻辑：
  if (NDI == 0) AND (DCI addressed to CS-RNTI):
    → Activation
  if (NDI == 1) AND (DCI addressed to CS-RNTI):
    → Deactivation
  if (HARQ process number matches configured CG):
    → Associated with this CG
```

#### 14.5.5 Multiple CG 场景下的调度优先级逻辑

当配置了多个 CG（最多 8 个 per BWP）时，MAC 实体的调度优先级逻辑：

```
Multiple CG Scheduling Logic:

优先级顺序（从高到低）：
1. CG Type 1（无 timer，可立即传输）
2. CG Type 2（已激活且 configuredGrantTimer 运行）
3. CG Type 2（已激活但 configuredGrantTimer 超时）

同一优先级的多个 CG：
  - 按 configuredGrantConfigIndex 顺序选择
  - 或由 UE 实现决定

Multiple CG + Dynamic Grant 冲突处理：
  ┌──────────────────────────────────────────────────────┐
  │ Grant A (dynamic, C-RNTI, URLLC LCH priority=1)     │
  │ Grant B (CG Type 1, eMBB LCH priority=8)              │
  │                                                      │
  │ lch-basedPrioritization 启用时：                     │
  │   → Grant A 优先级更高（priority 1 < 8）             │
  │   → Grant A 被传输，Grant B 降优先级                 │
  │   → Grant B 的 configuredGrantTimer 停止             │
  └──────────────────────────────────────────────────────┘
```

---

### 14.6 Packet Duplication 路径选择与恢复流程

#### 14.6.1 PDCP 层复制决策逻辑

```
PDCP Data PDU 到达 PDCP TX:

  ┌─────────────────────────────────────────────┐
  │  pdcp-Duplication = TRUE?                   │
  ├─────────────────────────────────────────────┤
  │ YES:                                        │
  │   → 复制 PDU，分别发送到每个 active RLC entity
  │   → Primary RLC Entity                      │
  │   → Secondary RLC Entity 1                   │
  │   → Secondary RLC Entity 2 (optional)         │
  │                                             │
  │ NO:                                         │
  │   → 正常传输到 primary RLC entity            │
  └─────────────────────────────────────────────┘

RLC Entity 激活状态决定复制数量：
  - Primary RLC: 始终存在且 active
  - Secondary RLC 1: 由 MAC CE T-bit = 1 激活
  - Secondary RLC 2: 由 MAC CE T-bit = 1 激活

复制路由示例：
  PDCP PDU → [Primary Path] RLC Entity 1 → MAC/Cell Group 1 → gNB-1
          └→ [Secondary Path] RLC Entity 2 → MAC/Cell Group 2 → gNB-2
```

#### 14.6.2 激活/去激活 MAC CE 格式与逻辑

```
┌─────────────────────────────────────────────────────────────────┐
│ Octet 0: [R(1)][LCID(5)][T(1)][R(1)]                          │
├─────────────────────────────────────────────────────────────────┤
│ R: Reserved (0)                                                │
│ LCID: 逻辑信道 ID（与 pdcp-Duplication 配置的 DRB 关联）        │
│ T: Duplication State                                          │
│   T = 1 → 激活该 LCID 的 duplication（add secondary RLC）      │
│   T = 0 → 去激活该 LCID 的 duplication（remove secondary RLC）  │
│ R: Reserved (0)                                                │
└─────────────────────────────────────────────────────────────────┘

激活流程：
  1. gNB 发送 MAC CE (LCID=X, T=1)
  2. UE PDCP 添加 Secondary RLC Entity (for this DRB)
  3. UE RRC 建立 secondary cell group 的 DRB 承载
  4. Secondary RLC Entity 进入 active 状态
  5. PDCP 开始复制到两个 RLC entities

去激活流程：
  1. gNB 发送 MAC CE (LCID=X, T=0)
  2. UE PDCP 停止复制到 Secondary RLC Entity
  3. Secondary RLC Entity 保持（但 idle）
  4. 如果 inactivityTimer 超时：释放 secondary RLC entity
```

#### 14.6.3 多路径传输失败后的恢复流程

```
路径失败场景与恢复：

Scenario A: Secondary Path 失败（Primary 正常）
  UE(MAC)                              gNB-1 (Primary)    gNB-2 (Secondary)
   │                                      │                    │
   │  PDCP PDU duplicated                │                    │
   │  → Primary RLC → PUSCH on Cell Group 1  │                   │
   │  → Secondary RLC → [LBT failure / ...]  │                   │
   │                                      │                    │
   │  Primary path HARQ-ACK = ACK         │                    │
   │  ◄─────────────────────────────────  │                    │
   │  Secondary path: 降优先级            │                    │
   │  Secondary RLC: pending             │                    │
   │                                      │                    │
   │  [Recovery] gNB 通过 CI-RNTI 取消 secondary?  │            │
   │  或 UE 自动停止 secondary 传输        │                    │
   ▼                                      ▼                    ▼

Scenario B: Primary Path 失败（Secondary 正常）
  UE(MAC)                              gNB-1 (Primary)    gNB-2 (Secondary)
   │                                      │                    │
   │  PDCP PDU duplicated                │                    │
   │  → Primary RLC → [PUSCH failure]    │                    │
   │  → Secondary RLC → PUSCH on Cell Group 2  │               │
   │                                      │                    │
   │  Secondary path HARQ-ACK = ACK      │                    │
   │  ◄───────────────────────────────────────────────────────  │
   │  Primary path: 自动降优先级           │                    │
   │                                      │                    │
   │  PDCP 收到来自 Secondary 的 PDU       │                    │
   │  → 交付上层（第一份到达）             │                    │
   │  → t-Reordering 处理                 │                    │
   ▼                                      ▼                    ▼

Scenario C: 两路径同时失败
  UE(MAC)
   │
   │  Primary + Secondary HARQ-ACK = NACK
   │  → gNB 调度 dynamic retransmission (C-RNTI)
   │  → 或 CG Retransmission Timer 超时后自主重传
   │  → 或 Slot Aggregation 重新传输
   ▼
```

#### 14.6.4 与 PDCP t-Reordering 的交互

```
PDCP 接收来自两个 RLC 路径的 PDUs：

  Path 1 (Primary RLC):     SN 5, 7, 9, 10 ...
  Path 2 (Secondary RLC):   SN 5, 6, 8, 9, 10 ...

PDCP RX state:
  RX_NEXT = 6 (等待 SN 6)
  RX_DELIV = 5 (已交付 SN <= 5)
  RX_REORDER = running (等待 SN 6)

t-Reordering 行为 (t-Reordering = ms2):

  Time T+0ms: SN 5 到达 (任一路径)
    → RX_DELIV = 5
    → SN 6 未到，start t-Reordering for SN 6

  Time T+1ms: SN 7 到达 (Path 1)
    → SN 6 仍未到
    → t-Reordering 继续运行

  Time T+1.5ms: SN 6 到达 (Path 2)
    → SN 5, 6, 7 现在连续
    → Deliver SN 5, 6, 7 to upper layers
    → Stop t-Reordering
    → RX_DELIV = 7

  Time T+2ms: SN 6 到达 (Path 1，延迟到达)
    → SN 6 已交付
    → Discard duplicate SN 6

关键点：
  - t-Reordering 确保 PDCP 交付给上层的顺序正确
  - 来自任一路径的第一份到达用于重组和交付
  - 延迟路径的重复 PDU 被丢弃
```

---

### 14.7 TDD HARQ-ACK 时序优化完整流程

#### 14.7.1 TDD 场景下的 HARQ-ACK 挑战

```
TDD DL/UL 配置问题：

配置：DL-heavy TDD (e.g., 8D + 1U + 1S)
  → 大部分时间 UE 只能接收，无法发送 PUCCH
  → HARQ-ACK 必须等到 UL slot
  → 等待时间可达 ~0.5~1 ms

传统解决方案（等待 UL subframe）：
  PDSCH (DL slot) → 等待 → PUCCH (UL slot)
  延迟增加：0.5~1 ms

URLLC 目标：0.5~1 ms 端到端延迟
  → 不能等待 slot 边界
  → 需要 symbol-level 的切换机制
```

#### 14.7.2 sSCell 配置流程

```
sSCell 配置（RRC）：

gNB                                    UE
 │                                      │
 │  RRCReconfiguration                 │
 │  ├─ scellToAddModList               │
 │  │   ├─ sCellIndex = 3             │
 │  │   ├─ cellIdentification          │
 │  │   │   └─ physCellId = 7         │
 │  │   ├─ sCellConfigToAddMod        │
 │  │   │   ├─ semistaticConfig       │
 │  │   │   │   └─ subframePattern   │
 │  │   │   │       = "UL slot in 5ms周期" │
 │  │   │   └─ dynamicIndication     │
 │  │   │       ├─ dci-Format = 1_1  │
 │  │   │       └─ switchingTime    │
 │  │   │           = sym1 (1 symbol)│
 │  │   │                              │
 │  │   └─ ul-Config                  │
 │  │       └─ ul-BWP                │
 │  │           └─ bwp-Id = 1        │
 │  │                                      │
 │  └─ RRCReconfigurationComplete     │
 │  ◄───────────────────────────────────│
 │                                      │
 │  UE 添加 SCell index 3               │
 │  sCell state: configured             │
 ▼                                      ▼
```

#### 14.7.3 切换方式一：半静态配置（Semi-Static）

```
半静态配置：RRC 配置周期性的 UL subframe pattern

sSCell 时域配置（每 5 ms 一个 UL subframe）：
  Subframe:  0   1   2   3   4   5   6   7   8   9
  PCell:     DL  DL  DL  DL  DL  DL  DL  DL  DL  UL
  sSCell:    UL  UL  UL  UL  UL  UL  UL  UL  UL  DL
                    ↑                               ↑

UE 行为规则：
  - PCell DL slot 时：
    → 如果 sSCell 配置了 UL subframe
    → 自动在 sSCell 上发送 PUCCH
  - 无需 MAC CE 或 DCI 指示
  - 切换延迟：~1 symbol (switchingTime)

优势：
  - 完全无信令开销
  - 可预测的切换模式
  - 适合周期性 URLLC 流量

限制：
  - 灵活性低（静态 pattern）
  - 可能与 eMBB UL 冲突
```

#### 14.7.4 切换方式二：动态 DCI 指示

```
动态 DCI 指示：通过 PDCCH DCI 1_1 指示 sSCell 切换

DCI 1_1 字段：
  "SCell indicator" (或 "PUCCH Cell Switch"):
    - 0: 使用 PCell 发送 PUCCH
    - 1: 使用 sSCell 发送 PUCCH

时序：
  ┌─────────────────────────────────────────────────────────────────┐
  │ DCI 1_1 received (PDCCH, PCell)                               │
  │   └─ SCell indicator = 1 (switch to sSCell)                   │
  │                                                                 │
  │ PDSCH received (PCell, DL slot)                               │
  │                                                                 │
  │ UE 解析 DCI：PUCCH 应在 sSCell 上发送                         │
  │                                                                 │
  │ t=0: SwitchingTime 消耗 (1 symbol)                            │
  │ t=1: PUCCCH transmission on sSCell (UL symbols)               │
  └─────────────────────────────────────────────────────────────────┘

优势：
  - 动态适应信道条件
  - 可根据 DL/UL 配置灵活切换
  - 适合非周期性 URLLC 流量

限制：
  - 需要 PDCCH 额外信令
  - DCI 解码延迟
```

#### 14.7.5 切换方式三：MAC CE 快速切换

```
MAC CE 切换：1 octet 快速指示

MAC CE 格式（SCell Switching）：
  Octet 0: [R(1)][R(1)][sCellIndex(5)][SwitchingState(1)]

  - sCellIndex: 切换目标 sSCell（1~31）
  - SwitchingState: 0=disabled, 1=enabled

时序：
  ┌─────────────────────────────────────────────────────────────────┐
  │ MAC CE received (sCellIndex=3, SwitchingState=1)              │
  │                                                                 │
  │ UE 执行 cell switching：                                        │
  │   - 停止 PCell PUCCH                    │
  │   - 启动 sSCell PUCCH                   │
  │   - switchingTime = 1 symbol            │
  │                                                                 │
  │ t=0: MAC CE 接收完成                                             │
  │ t=1: 切换完成，sSCell 可传输 PUCCH                              │
  └─────────────────────────────────────────────────────────────────┘

与 DCI 方式对比：
  ┌──────────────────┬─────────────────┬─────────────────┐
  │ 方式             │ 切换延迟        │ 信令开销        │
  ├──────────────────┼─────────────────┼─────────────────┤
  │ DCI 1_1          │ ~1~2 symbols   │ 1 DCI field     │
  │ MAC CE           │ 1~3 ms         │ 1 MAC CE (1 octet)│
  │ 半静态配置       │ 1 symbol       │ 0 (RRC only)    │
  └──────────────────┴─────────────────┴─────────────────┘
```

#### 14.7.6 与 PUCCH Group 的关联

```
PUCCH Group 概念（Rel-16）:

一个 UE 可以配置多个 PUCCH group：
  - Primary PUCCH Group: PCell 关联
  - Secondary PUCCH Group: sSCell 关联

HARQ-ACK 路由：
  - PCell 上配置的 PUCCH resources → PCell PUCCH group
  - sSCell 上配置的 PUCCH resources → sSCell PUCCH group

多 PUCCH Group 配置示例：
  PCell: PUCCH format 0/1/2 resources configured
  sSCell 1: PUCCH format 0 resources configured (URLLC dedicated)
  sSCell 2: No PUCCH resources (eMBB)

优势：
  - URLLC ACK 可在 sSCell 发送，与 PCell 隔离
  - 避免 DL-heavy TDD 下的等待延迟
  - 支持更灵活的 HARQ timing 管理
```

---

### 14.8 UE Capability 上报与网络配置交互

#### 14.8.1 UE Capability 询问/上报完整流程

```
UE Capability 交互流程（RRC）：

gNB                                    UE
 │                                      │
 │  UECapabilityRequest                 │
 │  ├─ requestCartesianProduct        │
 │  │   └─ bandCombination (requested bands) │
 │  └─ ue-CapabilityRequestFilter    │
 │  ◄──────────────────────────────────│
 │                                      │
 │  UE 评估自身能力：                   │
 │  ├─ 支持的 band combinations        │
 │  ├─ 支持的 feature sets            │
 │  ├─ URLLC 相关能力                  │
 │  │   ├─ mini-slot-based-scheduling │
 │  │   ├─ processingCapability1     │
 │  │   ├─ dmrs-Bundling             │
 │  │   ├─ urllc-Mode                │
 │  │   └─ urllc-UE                  │
 │  │                                  │
 │  │  如果 UE 不支持请求的能力组合：  │
 │  │  UE 上报最喜欢的 fallback 组合   │
 │  │                                  │
 │  UECapabilityInformation            │
 │  ├─ ue-CapabilityRats              │
 │  │   └─ nr (NR UE capability)     │
 │  │       ├─ ue-NR-Capability      │
 │  │       │   ├─ featureSets       │
 │  │       │   └─ ue-CapabilityCommon│
 │  │       └─ ...                   │
 │  └─ 非零长度的 UE Capability       │
 │  ──────────────────────────────────►
 │                                      │
 ▼                                      ▼
```

#### 14.8.2 URLLC 能力字段详解

```RRC
UE-NR-Capability ::= SEQUENCE {
  accessStratumRelease          ENUMERATED {rel15, rel16, rel17},
  pdcp-Parameters               SEQUENCE {
    ...
    ul-DataCompression          SEQUENCE {
      supported                  BOOLEAN,
      ...
    }  OPTIONAL,
    ...
  }  OPTIONAL,

  mac-Parameters               SEQUENCE {
    ...
    urlllc-UE                   BOOLEAN,     -- URLLC UE 标识
    multipleTCI                 BOOLEAN,
    multipleActiveConfig-indexingPerServingCell BOOLEAN,
    ...
  }  OPTIONAL,

  physicalChannelParameters    SEQUENCE {
    ...
    urllc-Mode                  SEQUENCE {
      mini-Slot-BasedScheduling  BOOLEAN,    -- 支持 mini-slot
      slot-Aggregation          BOOLEAN,    -- 支持 slot aggregation
      processingCapability1    BOOLEAN,    -- Processing Capability I
      dmrs-Bundling             BOOLEAN,    -- 支持 DMRS bundling
      typeB-Duration            SEQUENCE {
        duration2               BOOLEAN,
        duration4               BOOLEAN,
        ...
      }  OPTIONAL,
      ...
    }  OPTIONAL,
    ...
  }  OPTIONAL,
  ...
}
```

#### 14.8.3 网络根据 UE 能力决定配置

```
gNB 配置决策逻辑：

Step 1: 解析 UE Capability
  if (urllc-Mode.miniSlotBasedScheduling == true):
    → 可配置 Type B (mini-slot) 调度
  else:
    → 只能使用 Type A (slot-based)

  if (urllc-Mode.processingCapability1 == true):
    → K1 最小可设为 N1+1 = 9 symbols @ 15kHz
  else:
    → K1 最小需设为 N1'+1 = 14 symbols @ 15kHz

  if (urllc-Mode.dmrsBundling == true):
    → 可配置 dmrs-Bundling for slot aggregation
  else:
    → 不能使用 DMRS bundling

Step 2: 配置策略
  if (urllc-Mode == supported):
    → BWP-URLLC with SCS = 60/120 kHz
    → CG Type 1/2 with short periodicity
    → LCP Restrictions for URLLC LCH
    → PDSCH/PUSCH with Type B mapping
  else:
    → 使用 standard eMBB 配置
    → 告知应用层 URLLC 不可用

Step 3: Fallback 决策
  如果 UE 不支持 URLLC：
    → 通知应用层降级到 eMBB
    → 或使用增强的 eMBB 配置（降低码率提高可靠性）
```

#### 14.8.4 Capability Request/Response 交互序列图

```
gNB                                    UE                              时间
 │                                      │                               │
 │  UECapabilityRequest                 │                               │
 │  ◄──────────────────────────────────│ t=0                           │
 │                                      │                               │
 │                                      │ UE 评估能力                    │
 │                                      │ t=0~50ms                      │
 │  UECapabilityInformation             │                               │
 │  ──────────────────────────────────►│ t=50ms                        │
 │                                      │                               │
 │  gNB 配置决策：                       │                               │
 │  ├─ 配置 BWP-URLLC                   │                               │
 │  ├─ 配置 CG Type 1                   │                               │
 │  ├─ 配置 LCP Restrictions            │                               │
 │  └─ RRCReconfiguration              │                               │
 │  ──────────────────────────────────►│ t=100ms                       │
 │                                      │                               │
 │  UE 应用新配置                        │                               │
 │  UE ACK                              │                               │
 │  ◄──────────────────────────────────│ t=120ms                       │
 │                                      │                               │
 │  URLLC 配置完成                      │                               │
 ▼                                      ▼                               ▼
```

---

### 14.9 多连接数据路由与路径切换

#### 14.9.1 上行数据同时发送到两个 PDU Session

```
上行多连接（UL replication）路由：

UE                                    5GC
 │                                      │
 │  同一 PDCP Data PDU                  │
 │  ─► Duplicate                        │
 │      ├─ Path A → PDU Session 1       │
 │      │       └─ RLC Entity 1         │
 │      │           └─ Cell Group 1     │
 │      │               └─ gNB-1       │
 │      │                   └─ UPF-1    │
 │      │                                      │
 │      └─ Path B → PDU Session 2       │
 │              └─ RLC Entity 2         │
 │                  └─ Cell Group 2     │
 │                      └─ gNB-2       │
 │                          └─ UPF-2    │
 │                                      │
 │  两个路径独立传输                    │
 ▼                                      ▼

关键点：
  - 复制发生在 PDCP 层
  - 每个 PDU Session 有独立的 QoS Flow
  - 每个路径有独立的 RLC/MAC/PHY
  - 数据包内容完全相同
```

#### 14.9.2 接收端如何消除重复

```
DL 多连接去重（接收端 UE）：

gNB-1                                 gNB-2
 │                                      │
 │  同一 PDCP Data PDU                  │
 │  ├─ GTP-U N3 Tunnel 1              │
 │  │   └─ GTP-U N3 Tunnel 2          │
 │  │           │                     │
 │  ▼           ▼                     │
 │ UPF-1      UPF-2                  │
 │  │           │                     │
 │  └───────────┼───────────────────┘ │
 │              │ N3                    │
 │              ▼                      │
 │         NG-RAN                      │
 │              │                      │
 │      ┌───────┴───────┐             │
 │      │               │             │
 │  ┌───┴───┐      ┌────┴────┐       │
 │  │ gNB-1 │      │  gNB-2  │       │
 │  └───┬───┘      └────┬────┘       │
 │      │               │             │
 │      └───────┬───────┘             │
 │              │                     │
 │        PDCP duplication            │
 │              │                     │
 │              ▼                     │
 │           UE (PDCP RX)              │
 │              │                     │
 │  PDCP PDU SN=X arrives from Path 1  │
 │    → Stored in reordering buffer   │
 │    → t-Reordering started          │
 │                                      │
 │  PDCP PDU SN=X arrives from Path 2  │
 │    → Duplicate detected             │
 │    → Discard                       │
 │    → First copy delivered to upper │
 │                                      │
 ▼                                      ▼

去重机制：
  - PDCP SN (Sequence Number) 用于重复检测
  - t-Reordering timer 等待乱序到达的 PDU
  - 先到达的路径被标记为"winner"
  - 副本被丢弃
```

#### 14.9.3 路径失败时的切换逻辑

```
路径切换场景：

Scenario 1: Secondary Path 失败（Primary 继续）
  UE 检测到 Secondary path LBT failure 或 RLC failure:
    → RLC Entity 2 标记为 failed
    → PDCP 停止复制到 Secondary path
    → 只在 Primary path 传输（降级模式）
    → 配置去激活 MAC CE 通知 gNB
    → gNB 可决定是否重新激活或保持单路径

Scenario 2: Primary Path 失败（切换到 Secondary）
  UE 检测到 Primary path failure:
    → 立即切换到 Secondary path
    → Secondary path 成为 primary
    → PDCP 继续复制到新的 secondary（如果配置）
    → 上层应用无感知

Scenario 3: 两路径同时失败（降级）
  UE 检测到两路径均失败:
    → 触发 CG Retransmission Timer
    → 自主重传
    → 或触发 Scheduling Request 请求新 grant
    → 通知上层应用层"reliability degraded"

切换时序示例：
  ┌─────────────────────────────────────────────────────────────┐
  │ Time T: Secondary path LBT failure                          │
  │   → Secondary RLC entity = failed                          │
  │   → PDCP stop duplication                                 │
  │   → Primary path continues (no action needed)             │
  │   → gNB 收到 LBT failure indication MAC CE               │
  │   → gNB 可选择重新激活 Secondary 或保持单路径             │
  └─────────────────────────────────────────────────────────────┘

SMF 控制的路径切换（5GC 侧）：
  SMF 监控两个 UPF 的 QoS Flow 状态：
    → 如果 Path A 的 UPF-1 报告 failure
    → SMF 将流量切换到 Path B（UPF-2）
    → NG-RAN 被告知路径变更
    → UE 感知到配置更新
```

#### 14.9.4 多连接路径选择算法

```
多连接路径选择（gNB + 5GC 协同）：

Step 1: SMF 建立双 PDU Session（Redundant Session）
  - PDU Session 1: Primary path (UPF-1 → gNB-1)
  - PDU Session 2: Secondary path (UPF-2 → gNB-2)
  - 两个路径的 QoS Flow 指示：redundant transmission = true

Step 2: NG-RAN 配置 UE 的 Multi-CONNECTIVITY
  - RRCReconfiguration:
    ├─ DRB-ToAddModList
    │   └─ pdcp-Config: pdcp-Duplication = TRUE
    ├─ DRB-ToAddModList (Secondary)
    │   └─ pdcp-Config: pdcp-Config = secondary DRB
    └─ scg-Config: Secondary cell group

Step 3: 数据路由决策（per QoS Flow）
  每个 QoS Flow 有 redundancy indicator：
    - "no redundancy": 正常传输
    - "active redundancy": 双路径传输
    - "backup": 仅备用路径

Step 4: 动态路径切换
  gNB 监控：
    - path-specific BLER
    - HARQ-ACK success rate per path
    - LBT failure rate (unlicensed)

  如果 path-specific BLER > threshold：
    → gNB 可去激活该路径的 duplication
    → 通知 UE via MAC CE
    → 5GC SMF 收到路径状态更新
    → SMF 决定是否切换到单路径模式
```

---

## 十五、附录：关键规范索引

### 核心规范

| 规范编号 | 协议层 | URLLC 相关章节 |
|----------|--------|---------------|
| **TS 38.300** | NR Overview | Clause 16.1.1~16.1.8（URLLC）, Clause 16.8（TSC）, Clause 19（Coverage） |
| **TS 38.331** | RRC | Clause 5.6.1（UE Capability）, Clause 5.4.3（SR Config）, `ConfiguredGrantConfig`, `PDCP-Config`, `LogicalChannelConfig` |
| **TS 38.321** | MAC | Clause 5.4.3（LCP）, Clause 5.4.4（SR）, Clause 5.4.5（BSR）, Clause 5.8.2（CG Type 1/2）, Clause 6.1.3.31（Multiple CG CE） |
| **TS 38.322** | RLC | Clause 5.1（UM Mode）, Clause 5.3（t-Reordering）, Clause 5.4（Status） |
| **TS 38.323** | PDCP | Clause 4.3.1（Duplication）, Clause 5.1.2（t-Reordering）, Clause 5.1.5（Duplication procedures） |
| **TS 38.212** | PHY multiplexing | Clause 5.2.2（LDPC BG selection）, Clause 7.3（DCI formats） |
| **TS 38.213** | PHY procedures | Clause 4.2（UE processing timeline）, Clause 6.1.2（PDSCH allocation）, Clause 7.1（PUSCH allocation） |
| **TS 38.214** | PHY data | Clause 5.1.3（MCS selection）, Clause 5.2.2（CQI tables）, Clause 6.1.2/7.1（Time domain allocation） |

### 核心网络规范

| 规范编号 | 说明 | URLLC 相关章节 |
|----------|------|---------------|
| **TS 23.501** | 5GC 架构 | Clause 5.6.6（TSC）, Clause 5.6.8（Multi-connectivity）, Clause 5.6.8A（ATSSS）, Clause 5.7.1（PDU Session redundant） |
| **TS 23.502** | 5GC 流程 | Session 管理流程 |
| **TS 37.213** | Unlicensed | LBT operation for shared spectrum |

### TSC/时间同步

| 规范编号 | 说明 |
|----------|------|
| **TS 22.104** | TSC use cases and requirements |
| **TS 38.331** | TSN time reference configuration (SIBType31) |

---

## 十六、新增关键技术（Rel-18/5G-Advanced 前瞻）

> 📖 本章补充 Rel-15/16/17 之外的 URLLC 前沿技术，涵盖 DCI 1_2 紧凑调度、SPS DL 增强、PUSCH Repetition、端到端延迟预算、QoS 监控、网络切片与 CU/DU 分离架构，以及 Rel-18 5G-Advanced 增强方向。

### 16.1 DCI 1_2 紧凑型 DL 调度

#### 16.1.1 技术背景

DCI 1_2 是 3GPP Rel-16 引入的紧凑型 DL 调度格式（TS 38.212 Clause 7.3.1.2.2），与 DCI 0_2（UL 紧凑调度格式）配对使用。DCI 1_0 是常规 DL 调度格式，DCI 1_2 通过字段压缩实现更快的调度响应，适合 URLLC 高频小包调度场景。

**与 DCI 1_0 的主要区别：**

| 字段 | DCI 1_0 | DCI 1_2 | 压缩效果 |
|------|---------|---------|----------|
| Frequency domain resource | PRB 粒度 bitmap（完整频域） | 资源指示值（RIV）编码 | 字段长度大幅减少 |
| Time domain resource | 4-bit SLIV 索引 | 4-bit SLIV 索引 | 相同 |
| VRB-to-PRB mapping | 1 bit | 1 bit | 相同 |
| MCS | 5 bit | 5 bit | 相同 |
| RV | 2 bit | 2 bit | 相同 |
| DAI | 2 bit | 1 bit | 压缩 |
| TB beam ID | N/A | 新增字段 | URLLC 增强 |
| CSI request | 1~3 bit | 1~2 bit | 压缩 |
| PTRS-DMRS association | N/A | 新增字段 | 可靠性增强 |

#### 16.1.2 DCI 1_2 主要字段解析

```
DCI 1_2 字段结构（TS 38.212 Clause 7.3.1.2.2）：

+--------------------------------------------------+
| Identifier for DCI formats (1 bit, always '1')   |
+--------------------------------------------------+
| Hop flag (1 bit)                                 |
+--------------------------------------------------+
| Frequency domain resource assignment (variable)  |  ← RIV 编码，字段缩短
+--------------------------------------------------+
| Time domain resource assignment (4 bits)         |
+--------------------------------------------------+
| VRB-to-PRB mapping (1 bit)                       |
+--------------------------------------------------+
| Modulation and coding scheme (5 bits)           |
+--------------------------------------------------+
| Redundancy version (2 bits)                     |
+--------------------------------------------------+
| DAI (1 bit, for multi-VRB allocation)            |  ← 从 2 bit 压缩到 1 bit
+--------------------------------------------------+
| TB beam ID (0~3 bits, if configured)            |  ← Rel-16 新增
+--------------------------------------------------+
| CSI request (1~2 bits)                          |  ← 压缩
+--------------------------------------------------+
| PTRS-DMRS association (0~1 bit, if configured)  |  ← Rel-16 新增
+--------------------------------------------------+
| Padding bits (to align to DCI 0_2 size)        |
+--------------------------------------------------+
```

#### 16.1.3 使用场景与配置

DCI 1_2 适用于以下 URLLC 场景：

- **高频调度**：URLLC 业务到达间隔短，需要快速调度响应
- **小包传输**：TB size 较小，频域资源分配简单
- **低开销需求**：减少 DCI 开销，提升控制信道容量

**RRC 配置示例（简化版）：**

```
# DCI 1_2 的搜索空间配置（TS 38.331 Clause 5.6.1）
SearchSpace ::= SEQUENCE {
    searchSpaceId          INTEGER (0..39),
    controlResourceSetId   ControlResourceSetId,
    monitorSlotConfig      CHOICE {
        periodicityAndOffset  INTEGER (1..1600),
        numberOfContiguousSlots  INTEGER (2..80)
    },
    duration               INTEGER (1..16),
    candidateAggLevel-1    INTEGER (1..8),    ← DCI 1_2 通常配置较小聚合等级
    candidatesAggLevel-2   INTEGER (1..4),
    ...
}

# DCI format 1_2 由 SearchSpace 的 dci-Formats 指示
# dci-Formats ::= formats0-2   （配对 DCI 0_2 使用）
```

> ⚠️ **注意**：DCI 1_2 需要 UE 能力支持 `traffic-measurements` 或 `dci-format0-2-and-1-2`。网络侧通过 `UE-NR-Capability` 中的 `dci-Formats` 字段确认 UE 能力。

#### 16.1.4 与 DCI 0_2 的协同

DCI 1_2 与 DCI 0_2 共同构成 URLLC 紧凑调度体系：

```
URLLC 全紧凑调度模式：

  DL: DCI 1_2  →  PDSCH (compact)
  UL: DCI 0_2  →  PUSCH (compact)

配合使用效果：
  - 控制信道开销减少 ~30%（相比 DCI 1_0/0_0）
  - 调度响应时间缩短（字段更少，解析更快）
  - 适合 1~2 symbols 的 mini-slot 调度
```

---

### 16.2 SPS DL 增强与多配置

#### 16.2.1 SPS 与 CG 的对称性

在 URLLC 架构中，UL 有 Configured Grant（CG），DL 有 SPS（Semi-Persistent Scheduling）：

| 方向 | 机制 | 激活方式 | 规范引用 |
|------|------|----------|----------|
| **UL** | Configured Grant Type 1 | RRC 配置，直接使用 | TS 38.321 Clause 5.8.2 |
| **UL** | Configured Grant Type 2 | RRC 配置 + PDCCH 激活 | TS 38.321 Clause 5.8.2 |
| **DL** | SPS | RRC 配置 + PDCCH（CS-RNTI）激活 | TS 38.321 Clause 5.10.1 |

#### 16.2.2 SPS Configuration 结构

SPS 通过 RRC 配置，PDCCH 以 CS-RNTI（Configured Scheduling RNTI）加扰进行激活/去激活：

```
SPS-Config IE（TS 38.331 Clause 5.6.1）：

SPS-Config ::= SEQUENCE {
    n1PUCCH-AN          INTEGER (0..65535),
    # SPS DL 授权的 PUCCH 资源（HARQ-ACK 反馈）
    
    cqireporting         SetupRelease { CGI-ReportConfig }
    # 可选 CSI 报告配置
}

# DL SPS 授权配置（实际使用时由 activated DL SPS 配置决定）
ConfiguredGrantConfig ::= SEQUENCE {
    frequencyDomainAllocation      BIT STRING (SIZE(12)),  # UL 频域资源
    timeDomainAllocation           INTEGER (0..15),       # SLIV 索引
    frequencyDomainAllocation-r16  BIT STRING (SIZE(16)),
    ...
}

# 激活后的 SPS 参数（由 PDCCH 进一步指示）
# PDCCH 发送 CS-RNTI 加扰的 DCI，指示具体的 SPS 参数
```

#### 16.2.3 多 SPS 配置（Rel-16 增强）

Rel-16 引入多 SPS 配置支持，允许 UE 同时配置多个 SPS configuration，各自对应不同的业务流或 QoS 要求：

```
多 SPS 配置示例：

SPS-Config-List ::= SEQUENCE {
    sps-Config-ToAddModList ::= SEQUENCE {
        # SPS Config #1: URLLC 业务流
        SPS-Config ::= SEQUENCE {
            n1PUCCH-AN          0,
            periodicity         10ms,
            numberOfTransportBlocks  1,
            timeDomainAllocation    0,   # SLIV 索引
        },
        # SPS Config #2: URLLC 语音流
        SPS-Config ::= SEQUENCE {
            n1PUCCH-AN          1,
            periodicity         20ms,
            numberOfTransportBlocks  1,
            timeDomainAllocation    1,
        },
        # SPS Config #3: 高可靠 URLLC 控制
        SPS-Config ::= SEQUENCE {
            n1PUCCH-AN          2,
            periodicity         5ms,
            numberOfTransportBlocks  1,
            timeDomainAllocation    2,
        }
    }
}

# 每个 SPS 配置可配置不同的：
#   - 周期（5/10/20/40/80ms 等）
#   - 时域资源分配（不同 SLIV）
#   - 频域资源分配（不同频域位置）
#   - HARQ-ACK 资源（不同 n1PUCCH-AN）
```

#### 16.2.4 SPS 激活与去激活流程

```
SPS 激活流程：

1. RRC 配置 SPS
   UE 收到 RRCReconfiguration:
     - sps-Config IE 包含 periodicity、timeDomainAllocation 等
     - UE 进入 SPS 配置状态（待激活）

2. PDCCH 激活（CS-RNTI）
   gNB 发送 DCI（CS-RNTI 加扰）:
     - DCI 字段指示具体的 MCS、RV、时间偏移 K0
     - UE 解析 DCI 后进入激活状态

3. SPS 周期性传输
   UE 按配置的 periodicity 周期性接收 PDSCH
   无需每次收到 DCI 调度

SPS 去激活流程：
  gNB 发送 CS-RNTI 加扰的 DCI（toggleRV 或 Nouvelan）：
    - NDI = 0 且 RV = 0（对应新的 0 状态）
    - 或 UE 收到 explicit deactivation indication
```

#### 16.2.5 SPS 与 CG 的对称性总结

```
UL: Configured Grant (Type 1 / Type 2)
    - Type 1: RRC 配置，无 PDCCH 激活
    - Type 2: RRC 配置 + PDCCH 激活

DL: SPS (Semi-Persistent Scheduling)
    - RRC 配置 + PDCCH 激活（CS-RNTI）
    - Rel-16 支持多 SPS 配置

对称性优势：
  - URLLC 上下行链路均有免调度传输机制
  - 上下行均支持周期性资源预分配
  - 均可通过 PDCCH 动态激活/去激活
```

---

### 16.3 PUSCH Repetition Type A/B

#### 16.3.1 技术原理

PUSCH Repetition 是 Rel-16 引入的 UL 可靠性增强机制（TS 38.331 Clause 8.3），通过在同一 TB 的多次重复传输提升可靠性。与 Slot Aggregation（同一 TB 跨多个 slot 传输）的区别在于：PUSCH Repetition 每次传输都是 fresh encoding（新鲜编码），而 Slot Aggregation 是相同编码的跨 slot 重复。

**两种 Repetition Type：**

| 类型 | 别名 | 重复单位 | K 值 | 适用场景 |
|------|------|----------|------|----------|
| **Type A** | Slot-based repetition | Slot（14 symbols） | K consecutive slots | 连续 slot 重复 |
| **Type B** | Mini-slot based repetition | Mini-slot（1~14 symbols） | K repetitions（可跨 slot） | 灵活跨 slot 重复 |

#### 16.3.2 Type A：基于 Slot 的连续重复

Type A 在连续的多个 slot 上重复传输同一 TB：

```
PUSCH Repetition Type A 配置示例（TS 38.331）：

PUSCH-Config ::= SEQUENCE {
    repetitionType           ENUMERATED { typeA, typeB },
    pusch-RepTypeInfo-r16    SEQUENCE {
        nrofRepetitions       INTEGER (1..8),    # K 值
        timeDomainAllocation  INTEGER (0..15),   # SLIV
    } OPTIONAL
}

# 传输时序（Type A, K=4）：
# Slot N:     PUSCH 传输 #1
# Slot N+1:   PUSCH 传输 #2
# Slot N+2:   PUSCH 传输 #3
# Slot N+3:   PUSCH 传输 #4
#
# 每次传输均为 fresh encoding（独立 LDPC 编码）
# gNB 收到多个冗余版本后进行合并解调
```

#### 16.3.3 Type B：基于 Mini-slot 的重复

Type B 支持跨 slot 的 mini-slot 级别重复，更适合 URLLC 低延迟场景：

```
PUSCH Repetition Type B 配置示例：

PUSCH-RepTypeInfo-r16 ::= SEQUENCE {
    nrofRepetitions       INTEGER (1..8),
    timeDomainAllocation   SEQUENCE {
        mappingType        ENUMERATED { typeA, typeB },
        startSymbolAndLength   INTEGER (0..127),   # SLIV
    }
}

# Type B 传输示例（K=4, mini-slot 2 symbols）：
# Slot N, symbols 0-1:   PUSCH 传输 #1
# Slot N, symbols 4-5:   PUSCH 传输 #2
# Slot N+1, symbols 0-1: PUSCH 传输 #3
# Slot N+1, symbols 4-5: PUSCH 传输 #4
#
# 每次传输独立编码，gNB 合并增益更大
```

#### 16.3.4 与 Slot Aggregation 的区别

| 特性 | PUSCH Repetition | Slot Aggregation |
|------|------------------|------------------|
| 编码方式 | Fresh encoding（每次独立） | Same encoding（相同 TB） |
| 冗余类型 | 编码冗余（不同 RV） | 时间冗余（相同 RV） |
| K 值范围 | 1~8 | 2/4/8 |
| 调度粒度 | Slot 或 mini-slot | Slot |
| 可靠性增益 | 更高（编码增益+时间分集） | 中等（时间分集） |
| 规范引用 | TS 38.331 Clause 8.3 | TS 38.214 Clause 6.1.2 |

```
Slot Aggregation 传输示例（K=4）：
  Slot N:     PDSCH/PUSCH 传输（相同编码）
  Slot N+1:   PDSCH/PUSCH 传输（相同编码，RV=1）
  Slot N+2:   PDSCH/PUSCH 传输（相同编码，RV=2）
  Slot N+3:   PDSCH/PUSCH 传输（相同编码，RV=3）
  → 每次传输内容完全相同，只是 RV 不同

PUSCH Repetition 传输示例（Type A, K=4）：
  Slot N:     PUSCH 传输 #1（fresh encoding, RV=0）
  Slot N+1:   PUSCH 传输 #2（fresh encoding, RV=0）
  Slot N+2:   PUSCH 传输 #3（fresh encoding, RV=0）
  Slot N+3:   PUSCH 传输 #4（fresh encoding, RV=0）
  → 每次传输内容独立，gNB 合并后纠错能力更强
```

---

### 16.4 端到端延迟预算分析

#### 16.4.1 URLLC 1ms 延迟目标分配

3GPP URLLC 的设计目标是 **用户面延迟 ≤ 1 ms**（端到端，UE ↔ 网络）。这 1 ms 需要在各协议层和网络节点之间精细分配：

```
端到端延迟预算分配（理论最优值）：

┌──────────────────────────────────────────────────────────────────┐
│                          1 ms 总预算                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  UE 侧 PHY 处理：           ~71-143 μs                          │
│    - URLLC UE 处理能力 1：3GPP 规定 1 symbol                    │
│    - @ 60 kHz SCS: 1 symbol = 16.67 μs                          │
│    - @ 120 kHz SCS: 1 symbol = 8.33 μs                          │
│    - 加上混叠操作时间：~50-100 μs                                │
│    → 实际 PHY 处理：~71-143 μs                                   │
│                                                                  │
│  UE 侧 MAC+RLC+PDCP：     ~100-200 μs                            │
│    - MAC PDU 组装：< 10 μs（硬件实现）                          │
│    - RLC UM 封装：< 20 μs                                        │
│    - PDCP 加密 + 复制：< 30 μs                                   │
│    - 调度决策与 LCP 映射：< 50 μs                                │
│    → 实际 MAC+RLC+PDCP：~100-200 μs                              │
│                                                                  │
│  传输延迟（空口）：          ~500 μs（1 slot @ 60kHz）            │
│    - 1 slot @ 60 kHz = 0.125 ms = 125 μs                       │
│    - 1 slot @ 120 kHz = 0.0625 ms = 62.5 μs                    │
│    - 考虑 TTI 获取时间（1 slot）+ 处理时间                       │
│    → 实际传输延迟：~500 μs（保守估算）                            │
│                                                                  │
│  核心网延迟（UPF）：         ~200-500 μs                          │
│    - UPF 本地转发：< 50 μs                                       │
│    - 同城数据中心：< 200 μs                                       │
│    - 跨城核心网：> 1 ms（超出 URLLC 范围）                       │
│    → URLLC 要求：MEC/UPF 下沉到 RAN 边缘                         │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  总计：~0.87 ms ~ 1.04 ms                                        │
│  （理论最优，实际部署需预留调度开销）                              │
└──────────────────────────────────────────────────────────────────┘
```

#### 16.4.2 各层延迟详细分解

```
PHY 层延迟（UE 侧）：
  - UE 处理能力 Type 1（3GPP TS 38.214 Clause 4.2）：
    - PDSCH/PUSCH 处理时间：≤ 3 ODU（对于 URLLC，可配置为 1 ODU）
    - @ 60 kHz SCS: 1 symbol + 1 slot processing = ~142 μs
    - @ 120 kHz SCS: 1 symbol + 0.5 slot processing = ~71 μs
  - 处理能力 Type 2：
    - 支持更短的符号处理时间（~71 μs @ 60kHz）
    - 需要 UE 能力支持 `processingType2Enabled`

MAC 层延迟：
  - 调度请求处理：< 10 μs
  - LCP Restrictions 检查：< 5 μs
  - MAC PDU 组装：< 10 μs
  - HARQ 处理（无等待）：< 20 μs

RLC 层延迟：
  - UM 模式：无 ARQ 开销，< 20 μs
  - 分割/级联：< 15 μs

PDCP 层延迟：
  - 加密/解密：< 30 μs
  - 完整性保护：< 20 μs
  - 复制（Duplication）：< 10 μs

传输层（空口）延迟：
  - 1 slot @ 60 kHz = 125 μs
  - 1 slot @ 120 kHz = 62.5 μs
  - 考虑 HARQ RTT：URLLC 通常跳过 HARQ，使用 "no HARQ" 或 async HARQ
```

#### 16.4.3 延迟优化技术

```
延迟优化技术清单：

1. 免调度传输（Grant-Free）：
   - CG Type 1/2：消除调度请求 + DCI 等待时间（~500 μs）
   - SPS DL：消除 DL 调度 DCI 等待时间

2. Mini-slot 调度：
   - 1~2 symbol 传输：延迟从 1 slot 降至 1 symbol
   - @ 120 kHz SCS: 1 symbol = 8.33 μs

3. 处理能力增强：
   - UE Processing Capability 2：处理时间从 3 ODU 降至 1 ODU
   - 硬件加速：专用 URLLC 处理芯片

4. 跳过 HARQ：
   - URLLC 不使用同步 HARQ（等待 4 slots）
   - 使用异步 HARQ 或完全跳过重传

5. 核心网下沉：
   - MEC（Multi-access Edge Computing）部署
   - UPF 靠近 RAN 边缘
   - 核心网延迟从 ~5 ms 降至 < 0.5 ms
```

---

### 16.5 QoS 监控与延迟测量（TS 23.501）

#### 16.5.1 QoS Monitoring 架构

TS 23.501 Clause 5.7.7 定义了 5G 系统的 QoS Monitoring 机制，用于监控 QoS Flow 的延迟和丢包率：

```
QoS Monitoring 架构（TS 23.501 Clause 5.7.7）：

┌──────────────────┐         ┌──────────────────┐
│      UE          │         │       UPF        │
│  (DL measurement)│         │  (UL measurement)│
└────────┬─────────┘         └────────┬─────────┘
         │                            │
         ↓ DL                        ↓ UL
┌──────────────────┐         ┌──────────────────┐
│      gNB         │         │       gNB        │
│ DL packet delay  │         │ UL packet delay  │
│ measurement       │         │ measurement      │
└────────┬─────────┘         └────────┬─────────┘
         │                            │
         └──────────┬─────────────────┘
                    ↓
           ┌──────────────────┐
           │      AMF/SMF     │
           │  QoS monitoring  │
           │  control & storage│
           └──────────────────┘
```

#### 16.5.2 DL Packet Delay Measurement

gNB 负责测量 DL QoS Flow 的端到端延迟：

```
DL 延迟测量流程（TS 23.501 Clause 5.7.7）：

1. 测量点定义：
   - DL 延迟 = PDCP SDU 到达 gNB（RLC 入口） → UE 接收 PDSCH
   - 包含：RLC 排队 + MAC 调度 + 空口传输 + UE 处理

2. 测量触发：
   - 每个 QoS Flow 配置 QoS Monitoring 指示
   - 5QI 类型决定测量粒度（GBR / non-GBR）

3. 测量报告：
   - gNB 定期上报延迟统计（到 AMF/SMF）
   - 报告周期：可配置（100ms ~ 10s）
   - 指标：
     * Average DL delay
     * Max DL delay (P99)
     * DL packet loss rate

4. 超阈值告警：
   - 如果延迟 > threshold，gNB 触发 Notification
   - SMF 收到后可能触发 QoS Flow 重新协商
```

#### 16.5.3 UL Packet Delay Measurement

UPF 负责测量 UL QoS Flow 的延迟：

```
UL 延迟测量流程（TS 23.501 Clause 5.7.7）：

1. 测量点定义：
   - UL 延迟 = UPF 发送 IP packet → UE 发送 PUSCH
   - 包含：UPF 排队 + NG-U 传输 + gNB 处理 + 空口传输

2. 测量触发：
   - UPF 配置 UL QoS Monitoring
   - 5QI 类型决定是否启用

3. 测量报告：
   - UPF 定期上报延迟统计（到 SMF）
   - 指标：
     * Average UL delay
     * Max UL delay
     * UL packet loss rate

4. 与 DL 测量协同：
   - DL + UL 测量共同构成端到端 QoS 监控
   - 可用于 SLA 评估和计费
```

#### 16.5.4 URLLC QoS 指标要求

```
URLLC 5QI 对应的延迟/可靠性要求（TS 23.501 Clause 5.7.2）：

| 5QI     | 类型    | 延迟预算 | 可靠性    | 典型应用场景 |
|---------|---------|----------|-----------|--------------|
| 5QI=82  | GBR     | 10 ms    | 99.999%   | 远程控制      |
| 5QI=83  | GBR     | 10 ms    | 99.999%   | 触觉交互      |
| 5QI=84  | GBR     | 4 ms     | 99.999%   | 运动控制      |
| 5QI=85  | GBR     | 4 ms     | 99.999%   | 离散自动化    |
| 5QI=86  | GBR     | 1 ms     | 99.999%   | 智能电网配差   |
| 5QI=88  | GBR     | 10 ms    | 99.99%    | AR/VR        |
| 5QI=89  | GBR     | 10 ms    | 99.99%    | 云游戏        |

> 📖 规范来源：TS 23.501 Table 5.7.4-1（5QI 到 Packet Delay Budget 的映射）
```

#### 16.5.5 Packet Loss Rate Reporting

```
丢包率上报机制（TS 23.501 Clause 5.7.7）：

gNB / UPF 上报指标：
  - DL Packet Loss Rate：gNB 测量
  - UL Packet Loss Rate：UPF 测量
  - 测量周期：可配置
  - 上报对象：SMF / O&M 系统

URLLC 丢包率要求：
  - 99.999% 可靠性 → 丢包率 ≤ 0.001%
  - 对应 10⁻⁵ BLER（PHY 层）
  - 端到端丢包率（包含核心网）需更低

SLA 评估：
  - 延迟 + 丢包率 共同构成 URLLC SLA
  - 运维系统持续监控，触发告警
```

---

### 16.6 网络切片与 URLLC SLA

#### 16.6.1 5GC 网络切片架构

TS 23.501 Clause 5.15 定义了 5G 网络切片（Network Slice）架构：

```
网络切片架构（TS 23.501 Clause 5.15）：

┌─────────────────────────────────────────────────────────────────┐
│                        5GC (Service-Based)                       │
│                                                                  │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐          │
│  │  AMF    │   │  SMF    │   │  UPF    │   │  NEF    │          │
│  └────┬────┘   └────┬────┘   └────┬────┘   └─────────┘          │
│       │             │             │                               │
│  ┌────┴─────────────┴─────────────┴────┐                        │
│  │         Network Slice Instance      │                        │
│  │  (S-NSSAI = Slice Identifier)        │                        │
│  └──────────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
        ↑                       ↑
   PLMN (Public Land Mobile Network)
   SNPN (Self-Hosted Network)

S-NSSAI 结构：
  - SST (Slice/Service Type): e.g., URLLC, eMBB, mMTC
  - SD (Slice Differentiator): 额外区分参数
```

#### 16.6.2 URLLC 切片的 SLA 保障

URLLC 切片需要端到端的 SLA 保障，包括 RAN、传输网和核心网：

```
URLLC 切片 SLA 保障体系：

1. RAN 侧资源隔离：
   ┌────────────────────────────────────────────────────────────┐
   │  专属 BWP（Bandwidth Part）：                                 │
   │    - URLLC 业务使用独立 BWP，带宽 20/50 MHz                  │
   │    - 与 eMBB BWP 完全隔离                                    │
   │                                                              │
   │  专用调度资源池：                                             │
   │    - URLLC PUSCH/PDSCH 使用独立资源池                        │
   │    - 高优先级调度（LCP Restrictions）                        │
   │                                                              │
   │  专属 HARQ 进程：                                             │
   │    - URLLC 使用独立 HARQ 进程池                              │
   │    - 跳过 HARQ 或异步 HARQ                                   │
   └────────────────────────────────────────────────────────────┘

2. 核心网侧资源隔离：
   ┌────────────────────────────────────────────────────────────┐
   │  专属 UPF（用户面功能）：                                     │
   │    - URLLC 切片使用独立 UPF                                  │
   │    - UPF 下沉到 RAN 边缘（靠近基站）                         │
   │    - 减少传输延迟（< 0.5 ms）                               │
   │                                                              │
   │  专属 QoS Flow：                                             │
   │    - 5QI=82/83/84/85/86（GBR URLLC）                        │
   │    - gbr-qosFlowInformation：保障 GBR                       │
   │    - Priority Level：高优先级                               │
   └────────────────────────────────────────────────────────────┘

3. 传输网侧保障：
   ┌────────────────────────────────────────────────────────────┐
   │  专属承载（Dedicated Bearer）：                               │
   │    - 使用硬管道或软管道隔离                                  │
   │    - 低延迟转发（切片内优先调度）                             │
   │                                                              │
   │  端到端延迟预算：                                             │
   │    - RAN: ~500 μs（1 slot）                                 │
   │    - 传输网: < 200 μs                                       │
   │    - UPF: < 100 μs                                         │
   │    - 总计: < 1 ms                                          │
   └────────────────────────────────────────────────────────────┘
```

#### 16.6.3 工业互联网切片场景

```
工业互联网 URLLC 切片场景：

1. 工厂内网切片（Local Area）：
   - 部署模式：独立 UPF + 分布式 DU 靠近生产线
   - 延迟目标：< 0.5 ms（工厂内部通信）
   - 可靠性：99.999%
   - 典型应用：机器人控制、PLC 通信

2. 广域网切片（Wide Area）：
   - 部署模式：中心 UPF + RAN
   - 延迟目标：< 1 ms（端到端）
   - 可靠性：99.999%
   - 典型应用：远程操控、无人机控制

3. 混合切片：
   - 工厂内网 + 广域网协同
   - ATSSS 流量调度（3GPP TS 23.501 Clause 5.6.8A）
   - 本地流量本地处理，远程流量经广域网

切片管理流程（SMF）：
  1. SMF 接收 PDU Session 建立请求（S-NSSAI = URLLC）
  2. SMF 选择符合 SLA 的 UPF（靠近 RAN 边缘）
  3. SMF 配置 QoS Flow（5QI + GBR）
  4. SMF 监控 QoS 指标，持续保障 SLA
  5. 如果 SLA 降级，触发切片重配置或切换
```

---

### 16.7 CU/DU 分离架构对 URLLC 延迟的影响

#### 16.7.1 F1 接口延迟分析

3GPP 将 gNB 拆分为 CU（Central Unit）和 DU（Distributed Unit），F1 接口连接两者：

```
gNB 架构拆分（TS 38.401）：

┌─────────────────────────────────────────────────────────────────┐
│                           gNB                                   │
│  ┌───────────────────┐        ┌───────────────────┐            │
│  │        CU         │        │        DU         │            │
│  │  (RRC/PDCP 层)    │  F1    │  (RLC/MAC/PHY)   │            │
│  │                   │◄──────►│                   │            │
│  │  - RRC            │        │  - RLC            │            │
│  │  - PDCP           │ F1-C   │  - MAC            │            │
│  │  - SDAP           │ F1-U   │  - PHY            │            │
│  └───────────────────┘        └───────────────────┘            │
│         ↑                              ↑                        │
│         │                              │                        │
│    ┌────┴──────────────┐          ┌────┴──────────────┐        │
│    │     NG-RAN        │          │       UE          │        │
│    └───────────────────┘          └───────────────────┘        │
└─────────────────────────────────────────────────────────────────┘

F1 接口类型：
  - F1-C（控制面）：RRC、SCell 配置、F1AP 消息
  - F1-U（用户面）：RLC 数据传输、PDCP 状态

典型延迟：
  - F1-C 延迟：~1-2 ms（取决于部署距离）
  - F1-U 延迟：~0.5-1 ms
```

#### 16.7.2 对 URLLC 延迟的影响

CU/DU 分离会增加端到端延迟，对 URLLC 1ms 目标构成挑战：

```
CU/DU 分离对 URLLC 的影响分析：

场景 1: CU/DU 共置（传统部署）
  - F1 接口延迟：< 0.1 ms（几乎可忽略）
  - 对 URLLC 影响：无显著影响
  - 适用场景：常规 eMBB/URLLC 部署

场景 2: CU/DU 分离（中等距离）
  - F1 接口延迟：~1 ms
  - 对 URLLC 影响：
    * 用户面延迟增加 ~1 ms
    * 1ms 目标更难达成
    * 需要 uRLLC UPF 下沉 + 高效调度补偿

场景 3: CU/DU 分离（远距离，云化部署）
  - F1 接口延迟：~2-5 ms（跨城域）
  - 对 URLLC 影响：
    * 用户面延迟增加 2-5 ms
    * 超出 URLLC 1ms 目标范围
    * 仅适合非超低延迟 URLLC 场景

延迟预算重新分配：
  ┌────────────────────────────────────────────────────────────────┐
  │  URLLC 1ms 目标 + CU/DU 分离（1 ms 额外延迟）：                  │
  │                                                                │
  │  原有分配：                                                     │
  │    - UE PHY: ~0.1 ms                                           │
  │    - UE MAC+RLC+PDCP: ~0.2 ms                                  │
  │    - 空口传输: ~0.5 ms                                         │
  │    - 核心网: ~0.2 ms                                           │
  │    = 1.0 ms                                                    │
  │                                                                │
  │  引入 CU/DU 分离后（+1 ms）：                                    │
  │    - 需要重新分配或引入 edge UPF                                │
  │    - 方案：UPF 下沉到 DU 侧（near-RT RIC 级别）                  │
  │    - 核心网延迟从 0.2 ms 降至 < 0.1 ms                          │
  │    - 总计: 0.1 + 0.2 + 0.5 + 0.1 + 0.1 = ~1.0 ms              │
  └────────────────────────────────────────────────────────────────┘
```

#### 16.7.3 3GPP 解决方案

3GPP 为 CU/DU 分离架构下的 URLLC 提供了优化方案：

```
3GPP URLLC + CU/DU 分离优化方案：

1. Edge Computing UPF：
   - UPF 下沉到 RAN 边缘（靠近 DU）
   - F1-U 延迟可忽略（本地交换）
   - 核心网延迟从 ~1 ms 降至 < 0.1 ms

2. 分布式 DU 部署：
   - DU 靠近 RAN（工业区、工厂）
   - CU 集中部署在数据中心
   - F1 接口延迟可控（< 1 ms）

3. 紧耦合架构（E1 接口）：
   - E1 接口：CU-CP（控制面）↔ CU-UP（用户面）
   - 允许更灵活的部署选择

4. 延迟容忍设计：
   - 1ms 目标主要针对空口（UE ↔ gNB）
   - 核心网部分允许一定延迟（边缘计算补偿）

规范引用：
  - TS 38.401 Clause 6：gNB 架构与 F1 接口
  - TS 23.501 Clause 5.6.8：Edge Computing UPF 部署
  - TS 38.300 Clause 16.1.1：URLLC 延迟预算
```

---

### 16.8 Rel-18 / 5G-Advanced URLLC 增强

#### 16.8.1 Rel-18 URLLC 增强方向

3GPP Rel-18（5G-Advanced）继续推进 URLLC 增强，主要方向如下：

```
Rel-18 URLLC 增强方向（TS 38.300 Clause 9.2.8）：

1. AI/ML 辅助的 URLLC 调度：
   ┌────────────────────────────────────────────────────────────────┐
   │  基于 AI 的预测性调度：                                          │
   │    - 意图网络：gNB 基于历史数据预测 URLLC 业务到达               │
   │    - 资源预分配：在业务到达前预先配置资源                         │
   │    - 智能调度：AI 算法优化调度决策                               │
   │                                                                │
   │  规范进展：                                                      │
   │    - TS 38.300 Clause 9.2.8（Rel-18 URLLC enhancements）        │
   │    - TS 28.513：Intent-driven management for URLLC              │
   └────────────────────────────────────────────────────────────────┘

2. XR（Extended Reality）增强：
   ┌────────────────────────────────────────────────────────────────┐
   │  XR 业务对 URLLC 的新需求：                                      │
   │    - VR/AR：端到端延迟 < 10 ms，抖动 < 1 ms                      │
   │    - 云游戏：高可靠性 + 高带宽                                   │
   │    - 实时渲染：数据面低延迟                                      │
   │                                                                │
   │  Rel-18 增强方向：                                               │
   │    - 更好的 MAC 调度支持 XR 流量模式                             │
   │    - 增强的省电机制（XR 业务有 burst 特性）                      │
   │    - Multi-user XR 调度优化                                      │
   └────────────────────────────────────────────────────────────────┘

3. Further Enhanced LDPC BG2：
   ┌────────────────────────────────────────────────────────────────┐
   │  LDPC Base Graph 2 增强（TS 38.212 Clause 5.2.2）：             │
   │    - BG2 用于高可靠性 / 低码率场景                               │
   │    - Rel-18 进一步优化 BG2 的编码/译码性能                       │
   │    - 适用于 URLLC + eMBB 混合场景                               │
   └────────────────────────────────────────────────────────────────┘

4. Enhanced TSC for TSN 3.0 integration：
   ┌────────────────────────────────────────────────────────────────┐
   │  时间敏感网络（TSN）集成增强：                                    │
   │    - TSN 3.0：支持更多工业协议（Profinet、EtherCAT）               │
   │    - 更好的 time synchronization（10 ns 粒度）                    │
   │    - 5G TSN Bridge 增强                                          │
   └────────────────────────────────────────────────────────────────┘
```

#### 16.8.2 Rel-18 关键技术清单

```
Rel-18 URLLC 关键技术：

| 技术方向         | 规范引用         | 说明                                   |
|-----------------|------------------|----------------------------------------|
| AI/ML 调度      | TS 38.300 Cl.9.2 | 预测性资源分配，降低调度延迟            |
| XR 增强         | TS 38.300 Cl.9.3  | MAC 调度优化，省电机制                  |
| Enhanced LDPC   | TS 38.212 Cl.5.2  | BG2 性能优化                           |
| TSN 3.0         | TS 23.501 Cl.5.6.6| 工业协议集成                           |
| Multi-dimensional|                  |                                         |
| resource allocation|               | 多维资源调度（时/频/空/码域）            |
| Enhanced UE     | TS 38.331 Cl.5.6  | URLLC UE 处理能力进一步提升             |
| processing capability|              |                                         |
| Integrated sensing|                 | 通信+感知一体化（URLLC + 感知）          |
| and communication|                  |                                         |
```

#### 16.8.3 前沿研究方向

```
5G-Advanced URLLC 前沿研究方向：

1. 语义通信（Semantic Communications）：
   - 传输"意义"而非"比特"
   - 大幅降低传输数据量
   - 适用于 URLLC 控制消息

2. 触觉互联网（Tactile Internet）：
   - 端到端延迟 < 1 ms + 触觉反馈
   - 人机交互延迟 < 10 ms
   - 远程手术、远程操控

3. 全息通信（Holographic Communications）：
   - 超高带宽 + 超低延迟
   - 实时 3D 全息传输
   - URLLC + eMBB 融合场景

4. 通信-感知-计算一体化（ISAC）：
   - Integrated Sensing, Communication, Computing
   - URLLC + 环境感知融合
   - 智能工厂、智慧城市

5. 量子通信与安全：
   - 量子密钥分发（QKD）
   - URLLC 高安全需求场景
   - 金融、政务 URLLC 应用

规范跟踪：
  - 3GPP Release 18: URLLC enhancements（TS 38.300 Clause 9.2.8）
  - 3GPP Release 19: 5G-Advanced 第二阶段
  - O-RAN：AI/ML 在 RAN 中的应用（O-RAN WP）
```

---

> 📖 **规范来源**：TS 38.212 Clause 7.3.1.2.2（DCI 1_2）、TS 38.331 Clause 5.6.1（SPS-Config）、TS 38.331 Clause 8.3（PUSCH Repetition）、TS 23.501 Clause 5.7.7（QoS Monitoring）、TS 23.501 Clause 5.15（Network Slice）、TS 38.401 Clause 6（CU/DU）、TS 38.300 Clause 9.2.8（Rel-18 URLLC）

---

*文档生成时间：2026-05-29*
*检索工具：3GPP-KB（/opt/data/3GPP-KB/）*
*主要参考：3GPP Rel-17 TS 38.300、TS 38.321、TS 38.322、TS 38.323、TS 38.212、TS 38.213、TS 38.214、TS 38.331、TS 23.501*