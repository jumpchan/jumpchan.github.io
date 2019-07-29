---
title: '[Paper] HyCUBE: A CGRA with Reconfigurable Single-cycle Multi-hop Interconnect'
tags: [CGRA, Architecture, Interconnect]
categories: [Reconfigurable Computing]
mathjax: true
date: 2019-07-29 16:09:34
---

## 简介

第一篇内部连线可重构的CGRA架构。该架构在长距离FU之间提供单周期通信，因此不仅使得映射问题更高效，也取得比传统N2N（neighbor to neighbor）连接的CGRA三倍的效能功耗比（performance-per-watt）。

<!-- more -->

## 团队

新加坡国立大学的系列工作中，CGRA体系结构的研究。

## 相关工作

CGRA的传统连接方式大多是基于mesh，也就是邻居相连的结构。而进阶版也是多几个固定的跨越邻居的连线（mesh-plus），且连线方式是固定的。
关于mesh/multi-hop/ad-hop等概念，似乎是通信/无线网领域的技术，通过分布式节点构建完整的网络，因此用在了CGRA上。【具体还需要进一步调研】
而此前并没有关于CGRA内部连线的可重构设计工作。

## 贡献和动机

传统的长路径FU需要利用FU的布线功能来实现，这样不仅因为寄存器需要多周期，而且因为占用了FU作为布线资源，计算资源则相应减少，增加了映射限制。因此提供可配置的长路径布线功能则一方面维持简单的架构，保持低能耗，又提高了资源高效利用的可能性，提高性能，如下图例子中所示，d的性能优于c。

{% img https://qpimg.ml/images/2019/07/29/Mapping-of-DFG-on-N2N-CGRA-and-HyCUBE.png 500 Mapping of DFG on N2N CGRA and HyCUBE %}

- 提出新的结合了NoC的CGRA架构，该NoC拥有单周期多跳功能来为FUs创建虚拟动态的邻居，从而实现内部连线可重构；
- 提供基于新架构的应用映射问题，扩展MRRG来包含动态单周期路径；
- 在TSMC（台积电）28nm芯片上实现新架构，并与传统架构进行比较。

## 方法

该文提出的新架构，其网络拓扑结构仍然是mesh结构的，具体如下图所示。

{% img https://qpimg.ml/images/2019/07/29/A-4x4-HyCUBE-Architecture-with-a-single-cycle-multi-hop-multi-cast-path-between-tiles-scheduled-com--pletely-at-compile-time.png 600 A 4x4 HyCUBE Architecture, with a single-cycle multi-hop multi-cast path between tiles, scheduled com- pletely at compile time %}

允许某个FU在单周期访问多个远距离FUs的核心设计，就是在每个FU中增加一个6输入7输出的交叉开关（crossbar switch）。它管理来自四个方向的信号，使其要么使四个方向的信号异步地绕过（bypass），通过四个方向无时延中继器（clockless repeater），传递到下一个FU，或者停止信号传输，接收输入信号。而经过一个有SMART NoC的32nm处理器验证，无时延中继器能够在1ns内最多连接11跳。因此$4 \times 4$尺寸的HyCUBE上一般不会达到11跳的路径上限。[^Question:Size]

而在映射问题中，将该可重构设计在硬件抽象MRRG里表示出来则是主要工作，具体将中继器的所有可能的连线都作为一个布线资源的节点，表达在MRRG中，则可以抽象出所有的连接可能。如下图所示是一个$2 \times 2$的HyCUBE抽象MRRG，以及映射结果的展示。

{% img https://qpimg.ml/images/2019/07/29/MRRG-of-HyCUBE-with-single-cycle-routing.png 400 MRRG of HyCUBE with single-cycle routing %}

<!-- writing here -->

<!-- ![Alt_text](site "Title") -->
<!-- {% img site 500 Title %} -->

## 实验

该文用RTL实现$4 \times 4$HyCUBE架构，并在台积电（TSMC）28nm处理器上映射，用Synopsys Design Compiler作综合，用Cadence Encounter做布局布线。HyCUBE的编译器则作为LLVM 3.9的一个pass实现，用于将应用映射完成后对循环内核生成指令流。指令流则将被RTL模拟器用于评估能耗。而用于测试对循环内核是从MiBench和CortexSuite中挑选出来的。

作为比较，本文使用了两个基准CGRA架构，一个是标准的NoC（将无时延中继器换成有时延的），一个是N2N（传统CGRA）。具体各个指标的比较如下所示。

{% img https://qpimg.ml/images/2019/07/29/Quality-of-Mapping-MIIMapped-II.png 350 Quality of Mapping (MII/Mapped II) %}
{% img https://qpimg.ml/images/2019/07/29/Compilation-Time.png 350 Compilation Time %}
{% img https://qpimg.ml/images/2019/07/29/Performance-per-watt-w.r.t.-N2N-CGRA.png 350 Performance-per-watt w.r.t. N2N CGRA %}

而与其他加速器的比较如下图所示，HyCUBE可以很好地扩展到$4 \times 4$尺寸（相对比SRP的$3 \times 3$）；相比ARM（Cortex A5）能够提供3倍的性能和4倍的效能功耗比；相比FPGA（Artex 7）则主要在效能功耗比上具有明显优势，且对于相同应用，由于HyCUBE内部连线的可重构性，最终的映射结果更可能获得最佳的性能。

{% img https://qpimg.ml/images/2019/07/29/a-HyCUBE-chip-layout-bPower-Performance-comparison-with-commercial-processors-and-accelerators.png 350 (a) HyCUBE chip layout (b)Power-Performance comparison with commercial processors and accelerators %}

## 总结

HyCUBE首次提出功能单元之间的可重构的单周期多跳内部连线，允许单周期内布线，并基于此提出一个高效的编译器。

但该架构尺寸似乎文中仅扩展到$4 \times 4$，原因猜测为中继器的无时延路径不能过长导致的。而传统的NoC则加速没有那么高，不过传统NoC是不是也没有相关工作？因为本文实验中的StdNoC似乎是为了验证单周期特性而特别提出的架构。那么StdNoC可以作为大尺寸CGRA的解决方案？

[^Question:Size]: HyCUBE仍然是小尺寸低功耗处理器。$4 \times 4$的规模，是否因为中继器的原因不能扩展？

## Reference

> [1] Karunaratne, M., Mohite, A. K., Mitra, T., & Peh, L.-S. (2017). HyCUBE: A CGRA with Reconfigurable Single-cycle Multi-hop Interconnect. In Proceedings of the 54th Annual Design Automation Conference 2017 on - DAC ’17 (pp. 1–6). New York, New York, USA: ACM Press.  (Sites: [ACM][self]) [Back](#简介)

[self]: https://doi.org/10.1145/3061639.3062262 "[1] ${{ title }}"
