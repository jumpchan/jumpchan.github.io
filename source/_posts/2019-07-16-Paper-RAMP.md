---
title: '[Paper] RAMP: Resource-Aware Mapping for CGRAs'
tags: [CGRA, Mapping, Routing Strategy]
categories: [Reconfigurable Computing]
mathjax: false
date: 2019-07-16 09:58:59
---

## 简介

针对当前映射操作依赖到布线资源的策略，都是在调度后阶段，即P&R（Placement and Routing）阶段内调整布线方法，而没有重新调度全图，导致不能映射成功或者质量低下。
该文则提出在调度前针对三类未能映射的失败情况，在多个备选布线策略中进行选择，再依据新策略重新调度DDG，最后P&R。成功在映射质量、可映射数量以及求解速度方面得到全面提升。

<!-- more -->

## 团队

## 相关工作

现今各种的映射方法中，不同类型布线策略的最佳模型，均为备选策略。其中作为比较的，主要是寄存器感知和内存感知两种策略的模型。

## 贡献和动机

提出一种新的映射步骤，在原两步映射的前面，加入一个新的布线策略选择步骤。

- 该方式既能针对不同的映射失败情况，选择最能利用资源的布线策略，提高映射质量；
- 又能实现布线策略修改后的重新调度，从而提高映射成功率；
- 另外模拟通用编译器，新提出一个利用其他PE上分布式寄存器组来布线的策略，当该策略失败再选择利用内存来布线的策略，进一步提高质量。

总而言之，有点像一个集大成的工作，融合了现存各类映射方法。具体是以明确的选择方式，针对不同的情况，选择较优的策略为每个不能映射的依赖做重新映射，从而有针对性的避免各个方法的缺点，实现只取各类方法的优势，避免短板。

## 方法

针对每个不能映射的依赖，分析失败类型，分为以下三种：

1. 依赖的前置操作和后置操作被调度在长距离时间（distant timing）；而单个寄存器组不能映射过长的依赖；
2. 依赖的前置操作是一个正在操作的（live-in）或者只读（read-only）值；
3. 依赖的前置操作和后置操作被调度在结果时间（consequent timing）；因为连线资源不够或者空余PE不足，因此不能映射该依赖；

针对上述三种类型，分别选择下图中某个或者某些布线策略进行重新调度和P&R尝试，再在选择的策略的映射结果中选择最好的。

{% img https://qpimg.ml/images/2019/07/16/A-High-Level-Overview-of-RAMP.png 500 A High-Level Overview of RAMP %}

其中各个策略的简单示例如下图所示。

![Examples_of_Routing_Strategies](https://qpimg.ml/images/2019/07/16/Examples-of-Routing-Strategies-in-RAMP.png "Examples of Routing Strategies in RAMP")

<!-- writing here -->

<!-- ![Alt_text](site "Title") -->
<!-- {% img site 500 Title %} -->

## 实验

## 总结

## Reference

> [1] $Reference (Sites: [$Publisher][self]) [Back](#简介)

[self]: $site "[1] ${{ title }}"
