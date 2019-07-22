---
title: '[Paper] DNestMap: Mapping Deeply-Nested Loops on Ultra-Low Power CGRAs'
tags: [CGRA, Configuration Memory, Nested Loop, Cache, Partitioning, Packing, HyCUBE]
categories: [Reconfigurable Computing]
mathjax: false
date: 2019-07-18 21:56:43
---

## 简介

该文针对极低功耗CGRA中配置内存受限（很小）的情况，提出一个映射方法，映射包含深度嵌套循环和非规则控制流结构的应用。
该方法将应用和CGRA进行划分（partitioning），从多个嵌套循环中提取出最合适的代码块做映射，从而有效地通过时空（spatio-temporal）划分静态缓存这些划分后的代码块配置信息到配置内存中，以较少的启动间隔（II）代价，获得较多的配置内存加载时间降低，从而加快整体执行时间。

<!-- more -->

## 团队

新加坡国立大学的工作，从2017年DAC上的HyCUBE硬件架构，到2018年DAC上该论文关于映射深度嵌套循环到极低功耗CGRA上的工作。

## 相关工作

- 关于解决CMEM限制问题，有将配置内存当作缓存（cache），从时间上动态缓存配置信息的几种工作。
  - 包括cache prefetching、配置信息管理策略优化和多面体配置等工作；
- 关于该文方法的各个步骤的解决方法，涉及如下几种技术：
  - 三维正交背包问题（3D orthogonal knapsack, 3D-OKP）
  - 操作（Operation）的Packing Class抽象
  - 三维正交Packing问题(Orthogonal Packing Problem of 3-Dimension, OPP-3D)
  - 分支界限搜索（branch-and-bound search, B&B search）

## 贡献和动机

大多数应用包含嵌套循环，而当前大部分映射方法仅针对最内层循环，未考虑应用程序中复杂嵌套循环以及非规则控制流的部分如何映射。当映射该部分源码时，会对配置内存（Configuration Memory, CMEM）产生容量要求。针对超低功耗CGRA中CMEM极小时，嵌套循环用传统映射方法则会从片外内存频繁读取配置信息，在片上CMEM产生交换（swap）代价。

## 方法

【相关工作中的各个步骤的解决技术，需要进一步了解，以深入该文到底如何设计时空配置信息缓存方法的】
<!-- writing here -->

<!-- ![Alt_text](site "Title") -->
<!-- {% img site 500 Title %} -->

## 实验

## 总结

### 待商讨问题

- 针对MU到PE块的映射，用ILP求得局部最优解（此时尺寸都很小，求解应该很快），再做PEA划分等问题，能否提高质量？
- 既然整个运行时间包含很多breakdown的部分，能否用一种综合打分的机制，统一评价映射方法以及架构设计的好坏？

## Reference

> [1] Karunaratne, M., Tan, C., Kulkarni, A., Mitra, T., & Peh, L.-S. (2018). Dnestmap: Mapping Deeply-Nested Loops on Ultra-Low Power CGRAs. In Proceedings of the 55th Annual Design Automation Conference on - DAC ’18 (pp. 1–6). New York, New York, USA: ACM Press.  (Sites: [ACM][self]) [Back](#简介)

[self]: https://doi.org/10.1145/3195970.3196027 "[1] ${{ title }}"
