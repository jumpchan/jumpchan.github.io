---
title: >-
  [Paper] Recurrence Cycle Aware Modulo Scheduling for Coarse-Grained
  Reconfigurable Architectures
tags: [Modulo Scheduling]
categories: [Reconfigurable Computing]
mathjax: false
date: 2019-06-28 10:55:39
---

## 简介

该文基于EMS方法，将属于同一个重复环（recurrence cycle）【怎么翻译？】[^RecMII]的操作集中到一个聚合节点（clustered node）上，然后调度这些聚合节点。【这是类似Packing的思路吗？】
死锁（Deadlock）会在两个或者更多的recurrence cycle互相依赖时产生【形成环？】。使用启发式方法延长循环时延来解决死锁【具体怎么做的？】。
之前的工作需要权衡编译速度和结果质量，而该文的重复环感知方法则无须做这种权衡。

本文工作也是实现在他们自己的CGRA芯片和编译器解决方案中的，而调度结果的质量和速度均比基于模拟退火（simulated annealing, SA）的方法好。

[^RecMII]: 受在当次迭代中仍然需要依赖的上一个迭代中的操作对象（operands）影响，限制II取更小值的限制之一。

## 相关工作

- DRESC编译架构中使用基于模拟退火的模调度
  - 先初始化一个调度，此时仅考虑操作依赖，而不考虑资源冲突
  - 第二步调度器随机在FU之间移动操作，直到发现一个有效的调度
  - 该方法可以取得较好的结果，但是太耗时
- EMS集中在操作对象的布线，而不是操作的布局
  - 调度时间大量减少
  - 但调度质量没有DRESC好
  - 该文分析质量下降的主要原因是重复环中操作的投机调度
- 把操作集中起来再布局到FU中，并不是该文首创

从前人工作中，可以看出其idea一般源于两类方向，

- 一类是针对调度单个迭代时产生的问题，
  - 有方法层面的，例如增加计算资源利用率，包括：在调度时先集中操作再做布局，最后再布线；还有使用图着色算法等；
  - 也有硬件层面的，例如在调度中降低内存瓶颈、考虑连线时延等，包括建立操作树；将内部连线的时延和数量加入代价函数（cost function）；
  - 还有直接设计架构或者针对特定调度方法微调架构等。
- 另一类则是直接针对模调度算法
  - 引入模拟退火作为调度算法，但编译太耗时，耗费甚至可能超过几天的时间；
  - 在基于SA方法的中实现资源感知的布局方法，一定程度减少时间，但仍旧不能接受；
  - 使用经典图嵌入方法来映射DFG到硬件三维图表示中；
  - 以边为中心的模调度（edge- centric modulo scheduling, EMS）框架，集中在数据的操作对象（operands）的布线上，而不是数据的操作方式（operations）；

## 贡献

- 一个重复环感知的模调度技术，来移除调度中的非必须限制。
- 一个架构的微小改进，来显著提高调度质量。【软硬件协同设计？】

## 动机

EMS可以在局部对每个操作及其操作对象对布局布线产生快速有效对调度，但不能对全局的操作的调度顺序提供一个解决方案。现存EMS是基于操作在DFG中的高度（height）作为顺序的。而如果是调度一个重复环中的操作，则可能未考虑其生产者，先行局部调度。这就在调度中造成了非必须的约束，因为生产者需要与消费者在同一个II周期内，从而导致II的提升，影响调度质量。

换句话说，一旦一个重复环内的操作被调度，从而固定了调度时刻，则所有周期内的其他操作就被绑定在了该时刻。例如对下图中左边的数据流图映射到右边的架构中的过程进行调度。

{% img https://qpimg.ml/images/2019/07/01/example-of-recurrence-cycle-and-simplified-architecture.png 500 Example of recurrence cycle and simplified architecture %}

按DFG中高度来算优先级的话，会产生如下图的调度过程。没有输入的1，2，4，6是最高优先的，而仅有一个输入边的7也是最优先调度的操作之一。按该顺序开始调度，则会卡在下图的时刻4，因为没有多余的计算资源以供操作9使用。由于7被绑定在时刻3，重复环的时间限制则会要求操作10和11被调度到时刻4和5中。而操作9因为时刻3中没有可用的布线资源，也需要在时刻4中布局，和操作10分配仅剩的一个空余slot。

{% img https://qpimg.ml/images/2019/07/01/scheduling-process-is-stuck-by-recurrence-cycle.png 500 Scheduling process is stuck by recurrence cycle %}

## 重复环感知的CGRA映射

该方法的核心是一个智能操作排序策略，考虑重复环之间的关系，从而克服上述问题。

该方法先把重复环打包到一个聚合节点中，当作DFG中的单个节点。然后把其中的操作一起调度。对上述示例的打包如下图所示。该步骤其实对输入DFG进行了转换，形成了一个无环图，并重新计算高度，而该转换是解决上述问题的核心。

{% img https://qpimg.ml/images/2019/07/01/Group-recurrence-cycle.png 300 Group recurrence cycle %}

这样，重复环的所有生产者操作都会被先行调度，不管环中的操作优先级是不是更高。而当某个重复环代表的聚合节点的所有生产者都被调度后，无视优先级立即对聚合节点进行调度。重复环越早调度越好，因为此时资源尚未被太多占用。上述示例的调度结果如下图所示。

{% img https://qpimg.ml/images/2019/07/01/Successful-schedule.png 500 Successful schedule %}

## 硬件架构修改

修改RF的连接方式，如下图所示，从而增加值可传递的范围，增加了可调度的可能性，但也增加了长距离通信的时延，不过从结果看是有好处的。

{% img https://qpimg.ml/images/2019/07/01/CGRA-architecture-modification.png 300 CGRA architecture modification %}

## 实验结果

{% img https://qpimg.ml/images/2019/07/01/Comparison-of-the-scheduling-quality.png 700 Comparison of the scheduling quality %}
{% img https://qpimg.ml/images/2019/07/01/Comparison-of-the-compilation-time.png 700 Comparison of the compilation time %}

<!-- Writing Here -->
