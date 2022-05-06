---
layout: post
title: RISC-V CSR分析
categories: RISC-V CA
description: 对于RISC-V控制和状态寄存器的分析
keywords: RISC-V CSR
---

# RISC-V CSR分析

## 背景
RISC-V架构除了定义32个通用寄存器外，还定义了一些状态和控制寄存器（CSR, control and status register）。顾名思义，这类寄存器与控制CPU和表明CPU状态有关，是处理器内核内部的寄存器。CSR和常用的寄存器寻址的地址空间完全没有关系，使用其专有的12位地址编码空间，对一个hart（hardware thread，硬件线程），可以配置4K的CSR寄存器。RSIC-V架构中的CSR寄存器没有必要全部实现，实现部分也可，只要整个系统运行过程中不出错即可。同时可以进行自定义扩展CSR寄存器，毕竟CSR寄存器地址空间有12位