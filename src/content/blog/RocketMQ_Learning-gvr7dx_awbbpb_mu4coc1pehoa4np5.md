---
title: "Apache RocketMQ 创新论文被软件工程顶会 FM 2024 录用"
description: "Apache RocketMQ 创新论文被软件工程顶会 FM 2024 录用"
date: "2024-10-21"
category: "article"
keywords: ["RocketMQ_Learning"]
authors: "heimanba"
---

近日，由阿里云消息队列团队发表的关于 RocketMQ 锁性能优化论文被 CCF-A 类软件工程顶级会议 FM 2024 录用。

![](https://img.alicdn.com/imgextra/i1/O1CN01MhVYq41E39qBbxl6L_!!6000000000295-0-tps-1102-1664.jpg)

FM 2024 是由欧洲形式化方法协会（FME）组织的第 24 届国际研讨会，会议汇聚了来自各国的形式化研究学者，是形式化方法领域的顶级会议。FM 2021 强调形式化方法在广泛领域的开发和应用，包括软件、网络物理系统和基于计算机的综合系统。形式化方法以严格的数学化和机械化方法为基础来规约、构建和验证计算系统，是改善和确保计算系统质量的重要方法，其模型、技术和工具已延生成为计算思维的重要载体。

此次被录用的论文为《Beyond the Bottleneck: Enhancing High-Concurrency Systems with Lock Tuning》。此论文灵感来源于 RocketMQ 适配阿里云倚天 CPU 的性能优化过程中。RocketMQ 此前在发送消息的过程中存在两种锁：自旋锁和互斥锁。我们发现，不同 CPU 适合的锁行为并不相同。糟糕的锁行为可能导致性能的大幅下滑，而适配的锁行为能够在提升性能的同时降低资源损耗。这两种锁在版本迭代过程中，都在线上版本中使用过，且对于不同的版本来说，使用这两种锁可能带来截然不同的性能结果。

因此，**本文旨在提出一种新的自适应 K 值退避锁，能够让高并发系统的部署者无需考虑两种锁的优劣势，只需使用一把锁即可实现性能的最优以及最低的资源损耗。** 换言之，我们希望有一把锁能够同时具备自旋锁、互斥锁的特点，同时适用于竞争激烈和不激烈的情况。我们最终决定改造自旋锁，通过一把特殊的自旋锁，使系统在各种竞争情况下都保持非常优质的锁行为。自旋锁由于无限自旋直到获取到锁，在临界区较大时会产生较多的空转，耗费大量的 CPU 资源。为了能有效利用自旋锁的优势，因此我们要在临界区较大时对其空转次数的控制，从而避免大量空转，最大程度兼容临界区较大的场景。

最终，我们基于排队论，通过对自旋锁的行为建模，得到了自旋次数与系统负载的关系：

![](https://img.alicdn.com/imgextra/i3/O1CN01psLnaB1G6wLLHEBvk_!!6000000000574-0-tps-1448-314.jpg)

公式中，![](https://img.alicdn.com/imgextra/i4/O1CN01MiE8iK1s547dDYUTg_!!6000000005714-0-tps-316-116.jpg)是一把锁的整体期望获取时间。它分别由两部分组成：期望自旋耗时![](https://img.alicdn.com/imgextra/i3/O1CN01ssHb2B1m5cuYjowhp_!!6000000004903-0-tps-316-116.jpg)以及期望上下文切换耗时![](https://img.alicdn.com/imgextra/i1/O1CN01Zl61Kt29wyCmiE06g_!!6000000008133-0-tps-488-118.jpg)。将二者与自旋次数 K 和系统负载 P 的关系代入，则得到了上述的最终公式。公式中的 T<sub>s</sub> 是单次自旋耗时，T<sub>c</sub> 是单次上下文切换耗时。

我们最终基于系统的最大压力场景提出了自适应 K 值退避锁：**进行 K 次自旋后还未获得锁后，执行 Thread.yield() 将 CPU 执行权交给操作系统。** 这种行为能够避免互斥锁的无谓上下文切换，也能避免高压场景下的无限自旋带来的 CPU 损耗。这种行为能够缓解系统压力，取得自旋和 CPU 上下文切换两种方法中的最低开销。

![](https://img.alicdn.com/imgextra/i1/O1CN01I5iNt1227wlnxnM1d_!!6000000007074-2-tps-433-263.png)

在自适应 K 值退避锁的作用下，我们能找到系统性能的局部最优点，达到最大的 TPS 性能。结果如下表所示：

![](https://img.alicdn.com/imgextra/i3/O1CN01VjfDej1qsPYuSuvem_!!6000000005551-0-tps-1864-816.jpg)

<font style="color:rgb(136, 136, 136);">消息发送最大 TPS 的性能优化结果</font>

此外，我们还检查了各个 K 值下的 Broker 资源损耗情况，发现在最大 TPS 时的 K 值，同时也是资源占用相对最低时的 K 值：

![](https://img.alicdn.com/imgextra/i3/O1CN01HBlxDd1ju3fyJwU5p_!!6000000004607-0-tps-1744-806.jpg)

<font style="color:rgb(136, 136, 136);">各个 K 值下的 CPU 使用率</font>

以 X86 架构，同步刷盘的行为为例。实验结果表明，在 k= 10^3 时，发送速度不仅达到峰值（155019.20），CPU 使用率也达到最低。这表明退避策略成功地节省了 CPU 资源。此时，CPU 支持更高的性能水平和较低的利用率水平，这表明性能瓶颈已经转移——例如，可能已经转移到了磁盘上。在表中可以观察到，在具有相同的 k（10^3）和配置参数（最新代码，SYNC 刷盘模式）的 ARM CPU 上，RocketMQ 的性能提高了 10.4%。此外，如上图所示，当 k= 10^3 时，CPU 使用量大幅下降，从平均超过 1000% 下降到 750% 左右。资源消耗的减少表明，减轻其他系统瓶颈可能可以带来更显著的性能提高。

**<font style="color:rgb(255, 104, 39);">附论文信息</font>**

**录用论文题目：** 《Beyond the Bottleneck: Enhancing High-Concurrency Systems with Lock Tuning》

**作者：** 季俊涛，古崟佑，傅玉宝，林清山

**论文概述：** 高并发系统常常面临性能瓶颈，主要是由于线程间激烈竞争锁导致的等待和上下文切换。作为一家云计算公司，我们非常重视性能的最大化。为此，我们对轻量级自旋锁进行了改进，并提出了一种简洁的参数微调策略，能够在最低风险条件下突破系统性能瓶颈。该策略在高吞吐量消息队列系统 Apache RocketMQ 中得到了验证，实现了 X86 CPU 性能提升 37.58% 和 ARM CPU 性能提升 32.82%。此外，我们还确认了这种方法在不同代码版本和 IO 刷新策略下的一致有效性，显示出其在实际应用中的广泛适用性。这项工作不仅为解决高并发系统的性能问题提供了实用工具，还突显了形式化技术在工程问题解决中的实际价值。

**相关链接：**
[1] [FM 2024](https://www.fm24.polimi.it/)