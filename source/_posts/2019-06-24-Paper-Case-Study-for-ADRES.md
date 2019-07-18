---
title: >-
  [Paper] Design Methodology for a Tightly Coupled VLIW/Reconfigurable Matrix
  Architecture: A Case Study
tags: [CGRA, Case Study, ADRES, Modulo Scheduling, VLIW]
categories: [Reconfigurable Computing]
mathjax: false
date: 2019-06-21 10:47:44
---

## Introduction

[该文章][self]发表在2004年DATE会议上。主要工作是基于他们之前提出的模调度算法和对应的VLIW与CGRA紧密集成的新架构，提出的一个针对MPEG-2解码器应用的基于C的设计流案例，以此说明他们的设计方法能够完整映射一个应用到可重构系统中，并取得预期的堪比直接软件开发的性能。需要注意整个应用一般包含决定时间的规则化代码段和大量非规则的控制密集型代码，决定时间的往往是循环。考虑到应用代码的两个部分，一般硬件架构设计成由一个可重构阵列和一个处理器组成的架构（可将循环映射到阵列上加速，而控制密集型代码由处理器执行），处理器往往是一个RISC。其实这样的架构/系统设计是类似于软硬件协同设计问题的。

<!-- more -->

## Research Team

### Team Informations

该论文的主要研究者与2002年FPT上DRESC (Dynamically Reconfigurable Embedded System Compiler) 和2003年FPL上ADRES (Architecture for Dynamic Reconfigurable Embedded Systems) 的主要研究者是同一拨人。

### Previous Related Works

#### [一套编译系统：DRESC][DRESC]

【这是一套针对CGRA的编译系统？】[^Questions]

#### [一个新的架构：ADRES][ADRES]

这是一个超长指令字（Very Long Instruction Word, VLIW）和可重构阵列（reconfigurable matrix）紧密连接的架构模板（template）。区别于松散连接的精简指令集（Reduced Instruction Set Computing, RISC）和可重构阵列架构，ADRES取得了一些优势。【哪些优势？】[^Questions]

#### [新的模调度算法][Modulo]

2003年DATE的一篇文章，提出一个新的模调度算法，用于他们提出的ADRES架构。【是否集成在DRESC中？】[^Questions]

## 硬件架构：ADRES Architecture Overview

ADRES架构的整体系统结构如下图所示，类似于一个处理器，其执行核心（core）与【分级存储器？】[^Questions]（memory hierarchy）相连。
<!-- ![ADRES system](https://qpimg.ml/images/2019/06/21/ADRES-system.png "ADRES system") -->
{% img https://qpimg.ml/images/2019/06/21/ADRES-system.png 200 ADRES system %}

系统中的ADRES核心如下图所示，包含诸多组件，例如：【功能单元？】（Function Units, FUs）and 【寄存器组？】（register files, RF）。[^Questions]
<!-- ![ADRES core](https://qpimg.ml/images/2019/06/21/ADRES-core.png "ADRES core") -->
{% img https://qpimg.ml/images/2019/06/21/ADRES-core-with-dark-share-view.png 600 ADRES core %}

该核心有两个功能区域，这两个区域共享一些资源，如上图阴影部分所示，但因为【处理器/协处理器？】[^Questions]（processor/co-processor）模型，这两个区域在执行时永远不会在共享资源上有重叠。这两部分分别为：

- 超长指令字处理器：利用指令级并行（instruction-level parallelism, ILP）执行无计算内核（kernel）的代码。

这是一个典型的VLIW架构，分配一些FU，并用一个多端口（port）的RF将它们连接起来。而这些FU还能通过可用的端口与memory hierarchy相连。

- 可重构阵列：用高并行度的方式加速类数据流（dataflow-like）计算内核（kernel）。

除了与VLIW处理器共享的FU和RF部分，可重构阵列部分还有一些包含了FU和RF的可重构单元（reconfigurable cells, RC），其构造如下图所示。

<!-- ![Reconfigurable Cell](https://qpimg.ml/images/2019/06/21/Reconfigurable-Cell.png "Reconfigurable Cell") -->
{% img https://qpimg.ml/images/2019/06/21/Reconfigurable-Cell.png 350 Reconfigurable Cell %}

- FU可以是异构的，来支持不同的操作集。为了从循环（loop）中分离出控制流，FU支持预测操作（predicted operation）。
- 多路器（multiplexors）用于和其他资源数据直连。
- 配置存储器（configuration RAM）则本地（locally）存储一些【配置文本？】（configuration contexts）[^Questions]。
  - 配置环境可以在每个周期（cycle-by-cycle）从配置存储器中被循环读取。
  - 当本地配置存储器不够大时，配置环境也可以从memory hierarchy中读取，但会造成额外的延迟开销。
- 存储的共享（share）用于支持VLIW处理器与可重构阵列之间的通信，例如RF和memory hierarchy的共享。

ADRES的VLIW和可重构阵列紧密连接架构的优势：

1. 用VLIW能够加速非计算内核代码（non-kernel code），这些代码也往往是很多应用的瓶颈，而RISC不行；
2. 通过RF和内存存储的共享极大的降低了通信开销和编程复杂度；
3. 共享的资源能够极大降低【资源？】[^Questions]开销（costs）

## 基于C的设计流

ADRES的设计流如下图所示：

<!-- ![Design flow for ADRES](https://qpimg.ml/images/2019/06/23/Design-flow-for-ADRES.png "Design flow for ADRES") -->
{% img https://qpimg.ml/images/2019/06/23/Design-flow-for-ADRES.png 700 Design flow for ADRES %}

一个设计开始于使用C语言描述的应用，作为输入之一引入设计流。上图的右侧则是涉及目标硬件架构的基于XML的描述，作为另一个输入引入设计流。具体描述如下：

- profiling/partitioning：发现候选循环（loop），将在执行期映射到可重构阵列（P.S. 主要依靠人工识别）
- Source-level transformations：重构计算内核（kernel），以允许流水化
- IMPACT：是一个主要针对VLIW的编译器框架，用来语法解析、分析和优化C代码
- Lcode：IMPACT生成的一个中间表示，作为调度（scheduling）的输入
- Architecture：XML描述的语法分析和抽象，转化架构描述为内部图表示
- 接受应用程序和硬件表示作为输入，该工具分两个部分进行处理
  - 模调度（modulo scheduling）算法使得计算内核能够达到高并行度
  - 同时传统指令级并行调度使得非内核代码达到中度并行
- 这两部分的通信由工具发现并提供【是由register allocation部分负责吗？】[^Questions]
- 为VLIW和可重构阵列生成调度后代码，可由联合仿真器（co-simulator）进行仿真
- 另外一个正在进行的工作是内核调度，当配置存储器（configuration RAM）不能储存所有的内核时，该算法能够降低配置开销

一个中央配置文件（setting file）控制整个编译过程[^Question1]，包括整个设计环境和每个单独的循环。因此允许编译器在某一时刻集中关注单个循环，这种策略能降低设计时间，如下图所示。

<!-- ![Focus-on-one-loop-at-a-time.png](https://qpimg.ml/images/2019/06/23/Focus-on-one-loop-at-a-time.png "Focus on one loop at a time") -->
{% img https://qpimg.ml/images/2019/06/23/Focus-on-one-loop-at-a-time.png 700 Focus on one loop at a time %}

- 第一步，针对单个循环*loop1*，先进行源代码级转换输出*loop1\**，并由VLIW进行验证。这一步的用时极少；
- 第二步，则是对*loop1\**进行模调度，该文中的模调度算法类似一个布局布线算法，但仍然很耗时；
  - 模调度尝试找到一个调度，拥有尽量小的【启动间距？】（initiation interval, II）[^Questions]，即便II只有1的差距，也会造成很大影响；
  - 为了找到一个更小的II，值得耗时进行多次尝试，而如果模调度始终找不到好的解决方案，就返回源码级转换进行更多改进，直到找到一个合适的调度；
- 接着，暂存*loop1\**的映射到VLIW；
- 返回第一步迭代处理*loop2*，直到所有的循环均成果映射到可重构阵列上进行加速；
- 最后多个循环能被中央setting file整合起来，而不会发生特殊情况造成结果不确定。

这种设计策略的有点包括，能够使焦点专注于单个循环的映射，且每个循环能够被co-simulator验证，且对并行化设计流很有帮助。
而两个硬件部分VLIW和可重构阵列之间的通信，也因为紧密的硬件架构，变得能够依靠编译器自动化进行，且开销很小。仅需识别两部分需要产生和消耗的实时变量，并将其存入共享RF（VLIW RF）中即可，而通过内存空间的通信则更无须特别做什么，这两部分共享内存访问。

## MPEG-2解码器映射案例

MPEG-2解码器是一个多媒体领域的应用，需要很高的计算能耗。其大多执行时间消耗在若干计算内核（kernel）上，因此很适合用可重构架构来运行。该案例使用一个C实现的实例，源自MPEG Software Simulation Group (MSSG)。该实现的代码对处理器高度优化过，且是MediaBench的一部分。整个应用包含21个文件，共一万行左右代码。

### 映射到ADRES架构

- profiling：人工识别到应用中有14处循环（loop），作为可重构阵列的流水候选。另外，为了进一步提升性能，该文作者还从VLD(variable length decoding)循环中抽取出2个去量化循环，用于在可重构阵列中加速，如下图所示。所有16处循环占总执行时间的84.6%，却仅占总代码量的3.3%。

{% img https://qpimg.ml/images/2019/06/24/Extract-intra-dequant-loop.png 500 Extract intra dequant loop %}

- source-level transformation：除了少部分能直接映射的循环，其余需要先进行循环转换。
  - 例如下图中IDCT（inverse dis- crete cosine transformation）计算内核，其内层循环体，是一个函数，因此对于可重构阵列是不可流水化的。
    - 首先把循环体函数（图中的idctrow）的操作放入循环内
    - 接着把非规则计算（图中的shortcut）移除，使得循环更规则化
    - 最后把idct应用到一个微块上，使其符合硬件8x8的尺寸。对于MPEG-2视频一般使用6个8x8到块，也使得原来的8次迭代增加到64次迭代，极大降低了流水开支[^question2]。
  - 对于其他的不易流水化的循环，还有更多的转换方法，
    - 例如循环展开（loop unrolling）、【循环扁平化？】（loop flattening）、【变量复制？】（variable replicating）等[^Questions]
    - 但这些到目前（2004年）都需要根据设计者的经验来选择合适的。
    - 该文作者使用3天来完成应用程序中这些循环的C到C改写，实现循环转换。

{% img https://qpimg.ml/images/2019/06/24/Transformation-for-idct1-loop.png 500 Transformation for idct1 loop %}

当所有循环都被工具成功映射后，可以为每个循环获取其配置context。

而当MPEG-2应用到输入视频，改变大小时，循环数量会相应增加，导致配置存储器可能不够，因此该文正在解决的技术是内核调度技术。

### 映射实验结果

为了做实验，使用来一个类似MorphoSys的硬件实例作为ADRES模板的实现。
- 总共64个FU，被分为四片（tile），每片包含4x4的FU；
- 每个FU除了能连接相邻4个FU，还可以连接所有同一tile中的同行和同列FU，另外整个阵列还有行总线和列总线；
- 第一行FU能够被VLIW处理器使用，并连接到一个多端口寄存器组上；
- 只有第一行的FU能够执行内存操作，例如load/store操作。

实验结果如下图所示，第四列是Instruction-per-cycle（IPC）；第五列是total pipeline stages。第六列调度时间是在Pentium 4 1.7GHz/Linux PC平台上运行的CPU时间。

{% img https://qpimg.ml/images/2019/06/24/Scheduling-results-for-kernels.png 400 Scheduling results for kernels %}

【上述图示均为[文中][self]图片，有待重画】[^Questions]

[^Questions]: 用【】框起来的内容为尚未确定或者待解决的问题
[^Question1]: 【是不是存储在中央寄存器组中？因为该硬件与这个文件的属性很相符】
[^question2]: 流水开支是指？怎么降低流水开支的？

## Reference

> [1] Mei, Bingfeng, Serge Vernalde, Diederik Verkest, and Rudy Lauwereins. "Design methodology for a tightly coupled VLIW/reconfigurable matrix architecture: A case study." In Proceedings of the conference on Design, automation and test in Europe-Volume 2, p. 21224. IEEE Computer Society, 2004. (Sites: [IEEE][self]) [Back](#Introduction)
> [2] Mei, Bingfeng, Serge Vernalde, Diederik Verkest, Hugo De Man, and Rudy Lauwereins. "DRESC: A retargetable compiler for coarse-grained reconfigurable architectures." In 2002 IEEE International Conference on Field-Programmable Technology, 2002.(FPT). Proceedings., pp. 166-173. IEEE, 2002. (Sites: [IEEE][DRESC]) [Back](#一套编译系统：DRESC)
> [3] Mei, Bingfeng, Serge Vernalde, Diederik Verkest, Hugo De Man, and Rudy Lauwereins. "ADRES: An architecture with tightly coupled VLIW processor and coarse-grained reconfigurable matrix." In International Conference on Field Programmable Logic and Applications, pp. 61-70. Springer, Berlin, Heidelberg, 2003. (Sites: [Springer][ADRES]) [Back](#一个新的架构：ADRES)
> [4] Mei, Bingfeng, Serge Vernalde, Diederik Verkest, Hugo De Man, and Rudy Lauwereins. "Exploiting Loop-Level Parallelism on Coarse-Grained Reconfigurable Architectures Using Modulo Scheduling." In Proceedings of the conference on Design, Automation and Test in Europe-Volume 1, p. 10296. IEEE Computer Society, 2003. (Sites: [IEEE][Modulo]) [Back](#新的模调度算法)

[self]: https://dl.acm.org/citation.cfm?id=969178 "[1] Design methodology for a tightly coupled VLIW/reconfigurable matrix architecture: A case study"
[DRESC]: https://ieeexplore.ieee.org/abstract/document/1188678/ "[2] DRESC: A retargetable compiler for coarse-grained reconfigurable architectures"
[ADRES]: https://link.springer.com/chapter/10.1007/978-3-540-45234-8_7 "[3] ADRES: An architecture with tightly coupled VLIW processor and coarse-grained reconfigurable matrix"
[Modulo]: https://ieeexplore.ieee.org/document/1253623 "[4] Exploiting loop-level parallelism for coarse-grained reconfigurable architecture using modulo scheduling"
