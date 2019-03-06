---
title: Compiler Overview
mathjax: false
tags: [Compiler, Overview]
categories: [Compiler]
date: 2019-03-06 15:30:58
---

# Program Execution 程序执行

计算机程序执行过程主要涉及四个部分，包括[代码](#Code-代码)、翻译、中间表示和执行。其中侠义的编译器属于翻译部分，而一个完整的编译系统的执行包含了所有四个部分。

<!-- more -->

## Code 代码

代码（Code）是指由计算机程序的形式编码而成的一系列指令集合，它能够由计算机执行。在计算机硬件上运行软件需要两个部分，一个是代码（Code），另一个是数据（Data）。

计算机只能直接执行由指令集中指令构成的机器码（Machine Code）。但不论是机器码还是其他低级程序语言，都不是可以简单阅读的，因此大多源码（Source Code）都是由高级程序语言编写的。而编译器（Compiler）或者解释器（Interpreter）则将源码翻译成计算机可执行的机器语言。一个编译器生成的是目标代码（Object Code），一般来说就是机器语言（Machine Language）构成的，但也包括一些比源码更低级别的中间语言（Intermediate Language）。而解释器则是解释字节码（Bytecode）的工具，字节码是一种比源码低级的代码。

接下来具体说明代码的不同种类。

### Source Code

任意的使用人类可读的程序语言编写的代码组合都可称为源码（Source Code），它可能包含注释，一般使用富文本表达。源码一般会被汇编器（Assembler）或者编译器（Compiler）翻译成二进制机器码（Binary Machine Code），从而能够直接被计算机理解，这段机器码可能会被存储起来，从而晚一点再执行。除此之外，源码也可能被解释，从而立即执行。

### Object Code

目标代码（Object Code）是编译器的产物。通常来讲，目标码是一段状态或者指令的序列，它是由一种计算机语言表示的，一般是一种机器码语言或者中间语言，例如RTL (Register Transfer Language)就是一种中间语言。

### Bytecode

### Machine Code

### Microcode

## Translator (Computing)

### Compiler

#### Compile-time

### Optimizing Compiler

## Intermediate Representation (IR)

## Execution

### Runtime System

#### Runtime

### Executable

### Interpreter

### Virtual Machine 

---

# Compilation Strategies

## Just-in-Time (JIT)

### Tracing Just-in-Time

## Ahead-of-Time (AOT)

## Transcompilation

## Recompilation

[comment]: <> ([^脚注]: hi)
[comment]: ### ([^脚注]: hi)
[//]: (test)
[//]: test
