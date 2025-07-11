---
title: 802.11ax new feature list
date: 2025-06-06 07:53:36
tags:
    - [802.11ax]
    - [wifi6]
categories:
    - [ieee802.11, HE]
    - [ieee802.11, 11ax]
    - [ieee802.11, wifi6]
mathjax: true
---

# Part 1: Introduction

## 1. 更高的数据传输速率

- **更高阶的调制技术**：802.11ax采用最高1024-QAM正交幅度调制，相比802.11ac的256-QAM，每个符号可以传输更多的数据比特，从而提升了数据传输速率。

- **更长的FFT点数**：802.11ax将FFT点数增加到802.11ac的4倍，子载波间隔缩小至78.125kHz，这使得在相同带宽下可以传输更多的数据。

- **更高的理论最大速率**：802.11ax的最大理论速率可达9.6Gbps，相比802.11ac的6.9Gbps有了显著提升。


## 2. 更高效的多用户并发技术

- **OFDMA（正交频分多址）**：802.11ax引入了OFDMA技术，允许在同一个频段内将子载波分配给多个用户，从而提高了多用户并发的效率。

- **上下行MU-MIMO**：802.11ax不仅支持下行MU-MIMO（多用户多输入多输出），还新增了上行MU-MIMO，使得AP和多个终端之间可以同时进行数据传输，大大提升了多用户并发场景下的吞吐量。


## 3. 更强的抗干扰能力和覆盖范围

- **BSS Coloring（着色机制）**：通过在PHY报文头中添加BSS颜色字段，对来自不同BSS的数据进行标记，从而有效识别同频传输干扰信号，避免浪费无效的收发时间。

- **扩展覆盖范围**：802.11ax采用更长的OFDM符号发送机制（从3.2us提升到12.8us），并支持2MHz窄带传输，有效降低了终端丢包率和噪声干扰，提高了终端接收灵敏度，增加了覆盖范围。


## 4. 更好的节能特性

- **TWT（目标唤醒时间）**：允许AP对STA的唤醒与休眠进行统一调度安排，减少STA之间的冲突，减少STA不必要的唤醒次数，从而达到节能的目的。


## 5. 支持2.4GHz频段

802.11ax继续支持2.4GHz频段，这不仅有助于兼容老旧设备，还利用了2.4GHz频段覆盖范围广、穿透能力强的优势，适用于边缘区域覆盖补盲。

## 6. 其他改进

- **更大的帧聚合长度**：802.11ax支持更大的帧聚合长度（4194303字节），相比802.11ac的1048575字节，可以减少信道开销，提高传输效率。

- **空间复用技术**：802.11ax支持空间复用技术，进一步提升了频谱效率。


 802.11ax通过引入这些新技术和特性，不仅显著提升了数据传输速率，还优化了多用户并发性能、抗干扰能力和覆盖范围，同时兼顾了节能和兼容性，使其在高密场景和复杂环境中表现出色。


# Part 2: Details

1. FFT点数与子载波数量

FFT点数决定了OFDM符号中子载波的总数，点数增加意味着子载波数量增加。更小的子载波间隔使得在相同带宽内可以容纳更多的子载波，从而提高频谱效率。
以20MHz带宽为例，802.11ax的FFT点数为256，子载波间隔为78.125kHz，而802.11ac的FFT点数为64，子载波间隔为312.5kHz。在20MHz带宽下，802.11ax可容纳256个子载波，而802.11ac只能容纳64个子载波。

2. 更长的符号时间

子载波间隔缩小后，OFDM符号时间延长。例如，802.11ac的符号时间约为3.2微秒，而802.11ax的符号时间为12.8微秒。更长的符号时间使得保护间隔（GI）在总传输时间中的占比减少。
802.11ac的GI为0.8微秒，占用20%的时间。802.11ax的GI同样为0.8微秒，但在12.8微秒的符号周期中只占约5.9%。这提高了有效数据传输时间占比，从而提升数据传输效率。

3. 更好的抗多径干扰能力

更长的符号时间和更窄的子载波间隔提高了抗多径干扰的能力。在多径信道中，信号的延迟扩展可能会导致符号间干扰（ISI）。更长的符号时间使得相对延迟扩展更小，降低了ISI的影响。
保护间隔（GI）的设计也更有效，因为它在更长的符号周期中占比更小，进一步提高了数据传输效率。

4. 理论速率提升

速率提升与FFT点数、子载波间隔、调制方式等因素相关。假设调制方式为QPSK（2比特/符号），编码率为1/2，则：
802.11ac在20MHz带宽下的速率为：$速率=20×10^6 ×log_2(64) × 1/2 × 1/64 × 2=30Mbps$
802.11ax在20MHz带宽下的速率为：$速率=20×10^6 ×log_2(256) × 1/2 × 1/256 × 2=45Mbps$

通过这种方式，802.11ax在相同带宽下实现了更高的数据传输速率。

5. 频谱效率和数据传输效率

更长的符号时间和更窄的子载波间隔提高了频谱效率，使得802.11ax在相同带宽下能够传输更多的数据。通过这些优化，802.11ax在相同带宽下实现了更高的数据传输速率和更好的频谱效率。


## OFDM symbol time

### 关键参数对比

|参数|802.11ac|802.11ax|
|:--|:--|:--|
|FFT点数|64|256|
|子载波间隔|312.5kHz|78.125kHz|
|OFDM符号时间|3.2μs|12.8μs|
|保护间隔（GI）典型值|0.4μs or 0.8μs|0.8μs, 1.6μs or 3.2μs|

### 发送时间分析

- **802.11ac**：假设数据帧长度为N比特，调制方式为QAM，编码率为R，每个OFDM符号传输的数据比特数为 $N_{bit}^{AC}​=QAM比特数 \times 有效子载波数 \times R$ ，有效子载波数为52（20MHz带宽）。总符号数为 $N_{sym}^{AC}​=⌈\frac{N}{N_{bit}^{AC}}​⌉$ ，发送时间为 $T_{send}^{AC}​=(T_{sym}^{AC}​+T_{GI}^{AC}​) \times N_{sym}^{AC}​$。

- **802.11ax**：每个OFDM符号传输的数据比特数为 $N_{bit}^{AX}​=QAM比特数 \times 有效子载波数 \times R$，有效子载波数为234（20MHz带宽）。总符号数为 $N_{sym}^{AX}​=⌈\frac{N}{N_{bit}^{AX}}​⌉$ ，发送时间为 $T_{send}^{AX}​=(T_{sym}^{AX}​+T_{GI}^{AX}​) \times N_{sym}^{AX}​$​。


### 具体计算示例

假设数据帧长度为1000比特，调制方式为QPSK（每符号2比特），编码率R=1/2，GI分别为0.8μs：

- **802.11ac**：

    - 每个OFDM符号传输的数据比特数：2×52×1/2=52 比特。

    - 需要的OFDM符号数：⌈1000/52⌉=20 个。

    - 每个符号的总时间：3.2+0.8=4.0 μs。

    - 总发送时间：20×4.0=80.0 μs。

- **802.11ax**：

    - 每个OFDM符号传输的数据比特数：2×234×1/2=234 比特。

    - 需要的OFDM符号数：⌈1000/234⌉=5 个。

    - 每个符号的总时间：12.8+0.8=13.6 μs。

    - 总发送时间：5×13.6=68.0 μs。


### 结论

802.11ax的总发送时间为68.0μs，比802.11ac的80.0μs更短。这表明，尽管802.11ax的OFDM符号时间更长，但其更高的频谱效率（更多有效子载波数）使得总发送时间更短。