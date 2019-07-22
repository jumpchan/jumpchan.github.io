---
title: '[Paper] Architecture Exploration of Standard-Cell and FPGA-Overlay CGRAs Using the Open-Source CGRA-ME Framework'
tags: [CGRA, CGRA-ME, FPGA Overlay, Standard Cell, Physical Implementation]
categories: [Reconfigurable Computing]
mathjax: true
date: 2019-07-17 22:40:03
---

## 简介

Jason Anderson组CGRA-ME发布的首篇论文中，介绍其物理实现功能时，在ASIC上实现了Standard Cell CGRA。而该文则是继CGRA-ME发布后，扩充其物理实现功能，在FPGA上实现FPGA Overlay CGRA，并探索CGRA-ME如何在同一平台上比较CGRA的不同物理实现的性能、质量、能耗等指标。

<!-- more -->

## 团队

多伦多大学ECE部门的Jason H. Anderson组。

## 相关工作

针对CGRA-ME的三大功能：架构建模、应用映射和物理实现，分别有如下三种相关工作：

- CGADL建模语言，能描述CGRA架构，但没有实现应用的映射以及RTL输出；
- DRESC、SPR等映射系统，但没有集成到架构建模框架中；
- 各个硬件架构的物理实现工具也有很多工作，但都没有整合到一个大的统一框架中，以至于很难直接互相比较。

### 背景

#### Overlay

Overlay本身是网络的虚拟化技术，用在FPGA上则是虚拟平台，简而言之其框架如下三个部分：

- 应用：用HDL、C等实现的应用程序
  - 通过overlay mapping映射到虚拟平台
- Overlay：用CGRA等设计的虚拟平台
  - 通过物理映射和配置等映射到底层硬件上
- FPGA：包括LUT、RAM等组件

## 贡献和动机

似乎框架类论文的发表，都有系列论文产生的规律，从DRESC到CGRA-ME。这一系列包括首发论文介绍框架，案例论文介绍使用，有时还有众多其中创新组件的单独论文。
而该文就是CGRA-ME关于三大功能之一的物理实现的案例论文。

## 方法

该案例中CGRA-ME的架构如下图所示。需要注意的是数据流图是从源码中基于人工标注提取出来的。

![CGRA-ME_framework_overview](https://qpimg.ml/images/2019/07/17/CGRA-ME-framework-overview-showing-the-main-components.png "CGRA-ME framework overview showing the main components")

上图中CGRA-ME到三个主要功能包括：

- 从2的CGRA-ADL描述的架构到4输出的MRRG架构模型；
- 从1的包含人工标注数据流图DFG的应用源码到6的映射工具输出映射结果和配置流；
- 从2的CGRA-ADL描述的架构到7的RTL硬件描述语言生成，从而到模拟器或者硬件实现。

其中CGRA-ADL理论上能够描述任意CGRA架构；

映射模型则是能扩展到任意CGRA架构的整数线性规划（ILP）映射模型，能理论上产生最优解。虽然理论上映射速度最慢，但该文中针对所有Benchmark映射到$4\times4$尺寸CGRA的时间不超过12分钟。[^Question:大规模]

最后CGRA的物理实现一般有三种方式：定制芯片；一个standard-cell ASIC；在FPGA上作为一层Overlay。其中定制CGRA芯片的成本太高，因此该文实现并比较了后两种方式实现的CGRA。

<!-- ![Alt_text](site "Title") -->
<!-- {% img site 500 Title %} -->

## 实验

该文主要定义了基于ADRES架构模板的两个$4\times4$的变种架构实例，如下图所示，并对这两个CGRA实例在ASIC和FPGA物理硬件上进行了实现和比较。

{% img https://qpimg.ml/images/2019/07/18/22Full22-and-22Reduced22-variants-of-ADRES-architec--ture.png 500 "Full" and "Reduced" variants of ADRES architec- ture %}

### Standard-Cell CGRA

其中ASIC用的FreePDK45 standard-cell library，一个实验用途的开源45nm standard-cell库。而CGRA-ME生成的Verilog则用Synopsys‘ Design Compiler进行技术映射，具体分为最小面积（area）和最小时延（delay）两种映射；然后分析area breakdown，如下图所示；

{% img https://qpimg.ml/images/2019/07/18/Standard-cell-area-breakdown-of-Full-and-Reduced-architectures-when-targeting-minimum-area-and-minimum-delay.png 600 Standard-cell area breakdown of Full and Reduced architectures, when targeting minimum area and minimum delay %}

接着将映射设计netlist输入standard-cell P&R工具Cadence's Innovus，得到的P&R平面图如下图所示，输出SPEF文件和整个芯片面积（area），如下表第一列；最后用Synopsys' PrimeTime工具在设计netlist和SPEF文件基础上，进行静态时间分析（Static Timing Analysis, STA），得到芯片工作频率参数，如下表第二列。

{% img https://qpimg.ml/images/2019/07/18/Floorplan-for-Cadences-Innovus-place-and-route.png 400 Floorplan for Cadence’s Innovus place and route %}

 | Full Arch Min Area | Full Arch Min Delay | Reduced Arch Min Area | Reduced Arch Min Delay
 :-: | :-: | :-: | :-: | :-:
Area [μm2] | 317927.0 | 403753.2 | 253239.8 | 327189.2
FMax [MHz] | 140 | 164 | 142 | 192

而最后四种设计的硬件排布（layout）的物理视角和变形视角如下图所示。

{% img https://qpimg.ml/images/2019/07/18/Standard-cell-layout-for-all-four-designs-in-physical-view.png 300 Standard-cell layout for all four designs in physical view %}

{% img https://qpimg.ml/images/2019/07/18/Standard-cell-layout-for-all-four-designs-in-amoeba-view.png 300 Standard-cell layout for all four designs in amoeba view %}

### FPGA-Overlay CGRA

而FPGA的实验则用45nm的Altera Stratix IV FPGA芯片，在Altera/Intel的Quartus Prime 16.1上实现。而因为该FPGA上要实现CGRA的乘法器功能，只有通过DSP块。而由于该芯片DSP分布在某些列上，鉴于CGRA的设计需求，在硬件实现时DSP之间的LUT-based块将不会被充分利用。

在Stratix IV上实现两种架构，分析area breakdown得到下图，而统计的总体面积和性能参数如下表所示。其中面积的单位是ALMs（adaptive logic modules），每个ALM包含一个2输出可分割6输入LUT、一个进位链电路（carry-chain circuitry）和2个触发器（flip-flop）。另外包含乘法器（multiplier）的逻辑块使用Stratix IV中的DSP块，该面积并未列在表中统计。

{% img https://qpimg.ml/images/2019/07/18/FPGA-area-breakdown-for-both-architecture-vari--ants.png
 500 FPGA area breakdown for both architecture vari- ants %}

 | Full Arch | Reduced Arch
  :-: | :-: | :-:
Area [ALM] | 8803 | 7192
FMax [MHz] | 89 | 103

最后得到的FPGA实现硬件平面设计图如下图所示。

{% img https://qpimg.ml/images/2019/07/18/Floorplanned-Stratix-IV-implementations-of-ho--mogeneous-CGRA-with-torus-connections-left-and-hetero--geneous-CGRA-without-torus-connections-right.png
 500 Floorplanned Stratix IV implementations of ho- mogeneous CGRA with torus connections (left), and hetero- geneous CGRA without torus connections (right) %}

## 总结

CGRA-ME集合了三大功能于一身，包括架构建模、应用映射和物理实现，提供了一个统一的CGRA设计平台。[^Question:定制芯片]

[^Question:大规模]: 但未知大规模情况下如何，就如FPGA的VPR和Titan Benchmark运行时间上的差别，但相应但CGRA也需要有大规模的架构设计。
[^Question:定制芯片]: 该文物理实现案例中并未提供第一种实现方式，也就是在定制芯片上实现，不知此方法能否用GEM5实现？CCF是否使用的这种实现方式？

## Reference

> [1] Chin, S. A., Niu, K. P., Walker, M., Yin, S., Mertens, A., Lee, J., & Anderson, J. H. (2018). Architecture Exploration of Standard-Cell and FPGA-Overlay CGRAs Using the Open-Source CGRA-ME Framework. In Proceedings of the 2018 International Symposium on Physical Design - ISPD ’18 (pp. 48–55). New York, New York, USA: ACM Press.  (Sites: [ACM][self]) [Back](#简介)

[self]: https://doi.org/10.1145/3177540.3177553 "[1] ${{ title }}"
