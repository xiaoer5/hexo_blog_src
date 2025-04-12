---
title: 802.11be phy introduction
date: 2025-04-08 07:49:34
tags:
    - [802.11be]
    - [wifi7]
categories:
    - [ieee802.11, EHT]
    - [ieee802.11, 11be]
    - [ieee802.11, wifi7]
---

EHT PHY support:

- DL OFDMA
- UL OFDMA
- DL MU-MIMO
- UL MU-MIMO

<!--more-->

DL-MIMO and UL-MIMO are support as following conditions:
- entire PPDU bandwidth
- portions of the PPDU bandwidth, with assigned RU >= 242 tones

On a RU with MU-MIMO,

- max user number: up to 8 users

- spatial stream per user: up to 4 spatial stream

- total number of spatial streams: Not exceeding 8 for all users


EHT PHY Supports **preamble pruncturing** of an ETH MU PPDU for both OFDMA and non-OFDMA

ETH PHY defines RUs:

- 26 tones
- 52 tones
- 106 tones
- 242 tones
- 484 tones
- 996 tones
- 2x996 tones
- 4x996 tons

EHT PHY support an MRU assigned to single STA

- MRU: two or more RUs in certain combinations,

EHT PHY supports **GI** duration:

- 0.8us
- 1.6us
- 3.2us

EHT PHY supports **EHT-LTF** duration:

- 3.2us (1x)
- 6.4us (2x)
- 12.8us (4x)


!!! note What is ‌EHT-LTF?
    ‌EHT-LTF‌（EHT Long Training Field）：用于信道估计和同步的长训练字段。


!!! note 为什么需要不同的EHT-LTF符号持续时间?
    不同的持续时间可能对应不同的信道条件或系统配置。
    例如，
    较长的符号持续时间可能提供更好的信道估计精度，适用于复杂环境，
    较短的符号可能用于高吞吐量需求场景。

    同时，GI的持续时间也可能根据环境动态调整，例如在室内多径效应明显时使用较长的GI


!!! note what is **Symbol duration**
    **Symbol duration**：符号持续时间，指的是传输一个OFDM符号所需的时间长度，不包括保护间隔（GI）


!!! note what is **GI（Guard Interval）**
    **GI（Guard Interval）**：保护间隔，用于在多径传播环境中减少符号间干扰（ISI）。GI通常是符号持续时间的一部分。


!!! note what is **DFT period**
    **DFT period**：离散傅里叶变换周期，这应该与OFDM符号的有效数据部分相关，因为在OFDM中，DFT用于将频域信号转换为时域信号。DFT的长度决定了子载波的数量和符号的时间结构。


- 对于pre-EHT调制的字段，DFT周期是3.2微秒。

- 对于数据字段（Data field），DFT周期是12.8微秒。

**符号持续时间（排除GI）是否等同于DFT周期？**

在OFDM中，符号持续时间通常包括有效符号时间（即DFT周期）和保护间隔。
例如，在传统的802.11a/g/n中，符号持续时间是4微秒，其中3.2微秒是有效符号时间（DFT周期），0.8微秒是GI。所以，符号持续时间排除GI后的部分就是DFT周期。
因此，这里的描述应该是符号的有效部分（DFT周期）分别为3.2和12.8微秒。

**为什么pre-EHT字段和数据字段有不同的DFT周期？**

可能的解释是，为了向后兼容，pre-EHT字段使用与之前标准相同的参数，比如HE（802.11ax）可能使用3.2微秒的有效符号时间，而EHT的数据字段为了更高的效率或更大的带宽，采用更长的有效符号时间，比如12.8微秒。更长的DFT周期可能意味着更大的子载波间隔或更多的子载波数量，从而支持更高的数据速率或更宽的带宽

**什么是子载波间隔**

通常子载波间隔Δf = 1/T（T为有效符号时间）。

例如，如果T=3.2µs，Δf=1/3.2e-6 = 312.5 kHz，这是传统Wi-Fi的子载波间隔。
而如果T=12.8µs，Δf=78.125 kHz。更小的子载波间隔允许在相同带宽下更多的子载波，或者在更大带宽下保持子载波数量。
这可能与EHT支持的更大信道带宽（如320 MHz）有关，允许更长的符号时间，从而更有效地利用频谱，降低开销，提高效率。

针对 EHT PHY协议中 "The EHT PHY supports a symbol duration, excluding GI, a DFT period of 3.2 µs for the pre-EHT modulated fields and 12.8 µs for the Data field in an EHT PPDU" 这句话的理解:

> 在EHT的PPDU中，前导码中的pre-EHT部分（如L-STF, L-LTF等）使用3.2µs的有效符号时间（DFT周期），而数据部分则使用更长的12.8µs的有效符号时间。这样的设计既保证了与旧设备的兼容性，又允许数据部分采用更高效率的参数，提升吞吐量

**Pre-EHT字段**：PPDU前导码中沿用前代标准（如802.11a/n/ac/ax）调制的部分（如L-STF、L-LTF），确保旧设备能识别

**DFT周期**：完成离散傅里叶变换的时间窗口，决定了子载波间隔（Δf = 1/T）和频谱效率

!!! note **Pre-EHT字段的DFT周期（3.2 µs）**

    子载波间隔：Δf = 1/3.2µs = 312.5 kHz（与传统Wi-Fi一致）。

    **目的**：确保旧设备能正确解码前导码，维持向后兼容性。例如，前导码中的L-LTF字段需被所有设备识别以同步信道。


!!! note **Data字段的DFT周期（12.8 µs）**

    子载波间隔：Δf = 1/12.8µs = 78.125 kHz（更小的间隔）。

    优势：

    更多子载波：相同带宽下，子载波数量增加（如320 MHz带宽可容纳更多子载波），提升数据容量。

    抗多径干扰：更长的符号时间减少符号间干扰（ISI）的影响。

    开销优化：GI占总符号时间的比例降低（如12.8µs有效时间+0.8µs GI，GI占比仅5.9%，而3.2µs+0.8µs时占比20%），提升传输效率。



**data subcarrier frequency spacing**

ETH PHY data subcarrier frequency spacing is same as HE PHY  data subcarrier frequency spacing, And is 1/4 of VHT PHY and HT PHY  data subcarrier frequency spacing
(78.125 kHz = 1/4 * 312.5 kHz)


EHT PHY data subcarriers are modulated using:

- BPSK
- BPSK-DCM(EHT-mcs15)
- QPSK
- 16-QAM
- 64-QAM
- 256-QAM
- 1024-QAM
- 4096-QAM(ETH-mcs12 and mcs13)

!!! warning EHT-mcs15
    EHT mcs15 is **only** used in **single spatial stream** **none-MU-MIMO** transmission



**EHT DUP mode**

功能：通过重复传输相同数据（时域或频域）提升传输可靠性，适用于低信噪比（SNR）或高干扰场景。

设计背景：继承自Wi-Fi 6（802.11ax）的DUP OFDM模式，但针对EHT的高吞吐量和6 GHz频段特性优化。

**EHT MCS14**

定义：EHT Modulation and Coding Scheme 14（调制与编码方案14），是EHT新增的MCS索引，专为低速率、高可靠性场景设计。

关键参数：

调制方式：BPSK-DCM（Binary Phase Shift Keying with Dual Carrier Modulation）

编码方式：LDPC（Low-Density Parity-Check Code）

空间流数：单空间流（Single Spatial Stream, 1x1 SISO

**BPSK-DCM**

原理：将每比特数据映射到两个子载波（双载波调制），通过频域冗余提升抗频选衰落能力。

优势：

抗干扰能力增强（如多径效应、窄带干扰）。

适合低SNR环境（如边缘覆盖、工业物联网）。


**EHTMCS 14 vs. 传统MCS**

|**参数**|EHTMCS 14（DUP模式）|传统高吞吐量MCS（如MCS 11）|
|---|---|---|
|调制方式|BPSK-DCM|1024-QAM|
|编码速率|低（如1/2或1/4）|高（如5/6）|
|冗余机制|DUP模式 + 频域冗余|无冗余|
|目标场景|高可靠性、低SNR|高吞吐量、高SNR|
|典型吞吐量|极低（冗余牺牲速率）|极高（如2 Gbps以上）|
|适用设备|低功耗IoT、边缘设备|高端AP、智能手机|

#### **性能优势**

- **覆盖范围扩展**：通过冗余和低阶调制，传输距离可提升 30%~50%。

- **抗干扰能力**：BPSK-DCM 和 LDPC 编码在密集多径环境中表现优异。

- **法规合规性**：6 GHz 频段的功率限制下仍能保持可靠连接。

!!! note
    EHT MCS14 专为 6 GHz 频段的单空间流设备优化，平衡了速率与可靠性，拓展了 Wi-Fi 7 在工业、 IoT 等垂直领域的适用性。

