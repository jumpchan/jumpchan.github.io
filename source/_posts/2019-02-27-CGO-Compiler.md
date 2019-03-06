---
title: CGO Compiler
mathjax: false
tags: [Compiler, Record, Computer Architecture]
categories: [Conference]
date: 2019-02-27 17:00:00
---

This is a record of reading about working shop on CGO 19'. The workshop included talks from various projects - Julia (Julia Computing), TVM (UW), Glow (Facebook), XLA (Google), nGraph (Intel), TensorRT (Nvidia), and the soon to release MLIR (Google).

<!-- more -->

# Workshop on Compilers for Machine Learning ([C4ML][1])

## Julia: A Compiler to compile Code from Julia to XLA

> "Getting to Machine Learning from a General Purpose Compiler", Keno Fischer, Jameson Nash, [**Julia Computing**][2].
> Presentation: [PDF][3], [Blog][4]

这个编译器的目标是编译Julia代码到TPU平台的XLA代码上，因此编译器的Backend是LLVM。而LLVM是一个静态编译Backend，而Julia语言在语义上是动态语言，因此编译器需要转化原语言中的动态语义到LLVM的静态语义表示。

---

[1]: https://www.c4ml.org/ "Compilers for Machine Learning"
[2]: https://juliacomputing.com/communication/publications.html "Julia Computing Publications"
[3]: https://juliacomputing.com/assets/pdf/CGO_C4ML_talk.pdf "Julia Presentation"
[4]: https://juliacomputing.com/blog/2019/02/19/growing-a-compiler.html "Julia Blog"
