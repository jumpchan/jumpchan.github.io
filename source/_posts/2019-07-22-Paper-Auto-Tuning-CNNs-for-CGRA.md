---
title: '[Paper] Auto-Tuning CNNs for Coarse-Grained Reconfigurable Array-Based Accelerators'
tags: [CGRA, CNN]
categories: [Reconfigurable Computing]
mathjax: true
date: 2019-07-22 21:29:50
---

## 简介

2018年11月份发表的TCAD期刊论文。该文针对深度卷积神经网络（CNN）映射到不同尺寸SRP（三星商用CGRA）上的场景，提出一个自动微调编译器，依据神经网络结构源码和硬件架构特性，自动探索优化空间，生成优化后的代码，从而输入传统的映射方法完成网络到CGRA的映射。相对于传统直接映射，该文编译器能取得11倍的质量提升。

<!-- more -->

## 团队

首尔大学的硕士生做的研究，导师在三星做了三年工作。

## 相关工作

没有较通用的针对不同配置CGRA的CNN映射工作，前人的都是针对特定架构的CGRA，映射CNN。

## 贡献和动机

CNN中卷积层和全连阶层包含了大量循环迭代步骤，简单的循环体集中了大量上一层网络的数据和当前网络的权值之间的乘积操作。这样的应用很适合于用CGRA来进行硬件加速，但难点在于其多重嵌套循环，以及频繁的数据读写需求。
该文提出的自动化方法主要用于展开多重嵌套循环，使得计算密集型操作都展开并集中到最内层循环中；另外通过一些计算模式上的自动切换，尽量减少数据读写操作，进一步加速映射结果。

## 方法

架构如图所示，主要工作集中在源码转换，涉及模调度计算之前的所有代码分析和转换的自动化方法。

{% img https://qpimg.ml/images/2019/07/28/Optimization-steps.png 500 Optimization Steps %}

1和2部分选择卷积核的实现并生成每层网络的代码；包括网络训练、结构等设置相关的各个常量在3中发现并传递；4中则将激活函数的函数调用在最内层循环中展开；5则通过一个运行时时延估计模型来探索源码级转换技术的最佳组合，包括循环展开、循环合并和循环交换。最后在6中将最佳组合应用到抽象语法树上，并最终通过一个模调度器映射到CGRA上。

其中**卷积核实现**的选择需要考虑到卷积核尺寸和CGRA硬件尺寸及读写单元数量等因素，如下图所示，有两种较优的实现方式以供选择。

{% img https://qpimg.ml/images/2019/07/29/Computation-of-a-convolution-with-im2colGEMM-and-im2rowoptCGRA.png 450  Computation of a convolution with im2col+GEMM and im2row+optCGRA %}

显然第二种方式im2row+optCGRA能显著减少读写操作，利于循环展开，但如果CGRA的寄存器不足则可能需要外部存储单元的支持，第二种方式反而变得过于复杂。因此当CGRA寄存器能存储的数据量多于需要存储的数据量（$k \times k + k \times (k - 1)$）的两倍时，选择第二种方式，不然则用第一种方式im2col+GEMM。

而**循环展开自动化探索**则是通过找到最佳UF（Unrolling Factor）来实现，对于n层嵌套循环，则需要探索n个UF组成的向量，每层循环1个UF。特殊情况下，原循环（1到iter）通过UF展开，则会形成新的1到「iter/UF」的循环，每个迭代内包含原UF个迭代的所有操作，而剩下的「iter/UF」到「iter」的迭代则由另一个可展开的循环处理。具体探索过程从1到maxUF逐一尝试，该文设置maxUF为8，形象的处理过程如图所示。[^Question:UF]

{% img https://qpimg.ml/images/2019/07/29/Recursive-exploration-of-UFs.png 450 Recursive exploration of UFs %}

其中评估结果的好坏需要时延评估。最内层循环的时延的评估则是近似地根据最小启动间隔（MII）和迭代数量（n）相乘决定地。而嵌套循环的时延评估则需要考虑外层循环，通过MII与所有外层循环的迭代数（iter<sub>total</sub>）相乘得到。

该文最终实验量9种不同配置的CGRA，从$2 \times 2$到$4 \times 4$，如下图所示。

{% img https://qpimg.ml/images/2019/07/29/Screen-Shot-2019-07-29-at-15.53.51.png 500 CGRA overview and configurations %}
<!-- writing here -->

<!-- ![Alt_text](site "Title") -->
<!-- {% img site 500 Title %} -->

## 实验

该文比较了4种CNN在SRP（CGRA）上实现，与其他低功耗处理器的速度、能耗、工作频率等的比较，具体数值如下表所示。

{% img https://qpimg.ml/images/2019/07/29/COMPARISON-WITH-DIFFERENT-ACCELERATORS.png 700 COMPARISON WITH DIFFERENT ACCELERATORS %}

## 总结

- 该文中常量部分提到batch size的设置，但没有专门的训练优化？
该论文中似乎没提到针对训练进行优化。训练主要涉及频繁的参数更新，也就是内存/寄存器读写操作。
对于AIoT领域，或者其他嵌入式应用场景，虽然大多数时候是Inference较多，但是很多时候也涉及参数微调（迁移学习）操作。因此针对训练的优化也很重要。
- CGRA并不天然支持浮点数运算，让PE支持浮点数运算需要付出什么？
- 模调度的布线策略选择能否结合CNN进行？

[^Question:UF]: Unrolling Factor计算的相关工作是哪些？单层循环展开以及嵌套循环的循环展开区别和技巧有哪些？UF向量的思路其实是把嵌套循环的展开问题作为一个三维空间的搜索问题？

## Reference

> [1] Bae, I., Harris, B., Min, H., & Egger, B. (2018). Auto-Tuning CNNs for Coarse-Grained Reconfigurable Array-Based Accelerators. IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems, 37(11), 2301–2310.  (Sites: [IEEE][self]) [Back](#简介)

[self]: https://doi.org/10.1109/TCAD.2018.2857278 "[1] ${{ title }}"
