---
layout: post
title: 计算机体系结构基础-1
categories: CA
description: 计算机体系结构的基础知识
keywords: CA
---

# 计算机体系结构基础-1

为了系统性学习计算机体系结构知识，在这里对其基础内容进行一个总结。

## 1. 基本概念
### 1.1 指令级架构和微架构
指令级架构（Instruction Set Architecture, ISA）是CPU和软件之间的桥梁。ISA包括指令级、特权级、寄存器、执行模式、安全扩展、性能加速扩展等诸多方面。需要注意的是指令级架构和微架构（Microarchitecture）的区别，指令级架构是一个宏观、高层次的概念，规定的是CPU设计时一个大的框架；微架构也叫做计算机组织（Computer Organization），是规定CPU设计中的具体细节，是将一种给定的指令集架构在处理器中实现的方法。同一个ISA可以在不同的微架构上执行，同属于一个ISA的CPU可能内部的微架构差异很大。

### 1.2 超标量、多线程与多核
流水线CPU大大提高了性能，它的理想性能是每个时钟周期执行一条指令。超标量（Superscalar）CPU试图在一个周期内取出多条指令并行执行。由于指令之间的相关性，超标量CPU要增加大量的硬件电路来调度这些同时取出的指令，需要使用寄存器重命名、预约站、重排序缓冲等技术。超标量CPU对于性能提升已经很有限了，这是由于指令级并行性决定的，即使编译器可以使用诸如循环展开等技术。

多线程（Multithreading）CPU试图并行执行多个线程。一个线程就是一段能够独立执行的程序，再加上执行时所需要的环境。多线程CPU的特点是所有线程共享功能部件和cache，这有利于提高这些部件的使用效率，但增加了硬件设计的复杂度。多线程CPU试图通过线程级并行（Thread Level Parallelism, TLP）来提高CPU的性能。多线程的进阶版就是虚拟机。

**<font color=red>多线程和超标量之间的关系是什么？</font>**

多核（Multi-Core）CPU是在一个芯片中集成多个核，一个核就相当于一个普通的CPU，所有核可以共享第二级cache。多核CPU具有设计简单的优点，但功能部件的利用效率没有多线程CPU高。

多核CPU的每一个核可以是一个多线程CPU，这样的CPU可称为多核多线程CPU。一般来说，多核CPU中的每一个核都可以独立工作，这样的CPU可以实现多指令多数据流（MIMD）功能。若CPU中由多个处理单元PE（Processing Element），所有的PE执行相同的指令，对多个数据同时做同样的运算，这样实现的就是单指令多数据流（SIMD）功能。**所以实现SIMD功能的关键在于有多个相同的PE硬件，可以对多个数据同时做同样的处理。**

体系结构改变的方向：流水线，超标量，多线程，多核


### 1.3 高性能计算机和互联网络
高性能计算机是指那些包含有多个CPU和多台计算机的系统，主要分为两类：多处理机（Multiprocessors）系统和多计算机（Multicomputers）系统。前者共享存储器，又称为并行系统（Parallel Systems）；后者不共享，称为分布式系统（Distributed Systems）。超级计算机（Supercomputers）和服务器（Servers）一般是共享存储器的并行系统，而用于网格计算（Grid Computing）以及云计算（Cloud Computing）的环境由分布式系统构成。互联网络（Interconnection Network）在并行系统中占有重要位置，它连接所有的CPU，CPU与远程存储器之间的数据传输要经过它才能进行。

## 2. 指令系统
### 2.1 历史演化
根据指令长度的不同，指令系统可以分为复杂指令集（CISC）、精简指令集（RISC）和超长指令字（VLIW）。CSIC中的指令长度可变；RISC中的指令长度比较固定；VLIW本质上是多条同时执行的指令的组合，同时执行的特性由编译器指定，无需硬件进行判断。

早期CPU都是CISC结构，指令复杂，这种设计简化了软件和编译器的设计，但显著提升了硬件的复杂度。

RISC简化了访存类型，访存只能通过load/store指令实现，这种实现方式，简化了指令间的关系，让所有运算指令都是对寄存器运算，所有访存都通过专用的访存指令进行。CPU只要比较寄存器号即可判断运算指令之间以及运算指令和访存指令之间有没有数据相关性，简化了数据相关性判断的复杂度，有利于CPU实现指令流水线、多发射、乱序执行等提高性能。

VLIW的最初思想是最大限度利用指令级并行，VLIW的一个超长指令字由多个互相不存在相关性的指令组成，可并行进行处理，其简化了硬件实现，但增加了编译器的设计难度。VLIW在DSP和GPU上还有应用，现在在AI芯片设计上也有一些应用。

### 2.2 指令系统组成
#### 2.2.1 地址空间
#### 2.2.2 操作数
#### 2.2.3 指令操作和编码
#### 2.2.4 特权指令和非特权指令


## 3. 异常和中断



## 4. 并行性
### 4.1 指令级并行
是20世纪最后20年体系结构提升性能的主要途径，主要分为两种。一是时间并行，即指令流水线。二是空间并行，即多发射（multi-issue），或者叫超标量（superscalar）。多发射就像是多车道的马路，而乱序执行（Out-of-Order Execution）就是允许在多车道上超车，超标量和乱序执行常常一起使用来提高效率。2010年后进一步挖掘指令级并行的空间已经不大了。

超标量/多发射是具有多个功能部件，是硬件基础；而乱序执行是可以在多个功能部件中自由执行指令，是一种技术手段，二者是不同的概念，并且结合起来才能提升效率。

### 4.2 数据级并行
主要是指单指令多数据流（SIMD）的向量结构。SIMD作为指令级并行的有效补充，早期主要是用于专用处理器上，现在已经成为通用处理器的标配。

### 4.3 任务级并行
大量存在于网络应用中，其代表是多核处理器以及多线程处理器，是目前计算机体系结构提升性能的主要方法。任务级并行的粒度较大，一个线程中可包含几百条甚至更多的指令。


### 4.4 小结
上述三种并行方式在现在处理器中都有，多核处理器运行线程级或进程级并行的程序，每个核采用多发射流水线结构，而且往往有SIMD向量部件。

在这里简述一下超标量处理器和多核处理器的区别。超标量处理是在指令级并行（ILP）层面来发挥并行性，通过乱序执行，多发射等手段，来解决指令之间的数据相关性，在处理器内部设置多条流水线，使得处理器内部有尽可能多的指令在执行。这种并行性，对外不可见。超标量处理器内部只有一个指令指针，一套控制逻辑，对于外界而言，在同一时刻，只能执行一段指令序列。多核处理器是在线程级并行（TLP）层面解决问题，这种并行是对外可见的，现代处理器单个核心一般可以支持多线程执行，所以处理器同时可以执行的线程远大于物理核心数，在多核处理器中，各个核心被看做是独立的处理器，共享同一个内存，不同的核心之间执行不同的指令，每个核心的PC，寄存器，控制逻辑都是独立的。多核处理器的核心可以设计为超标量架构。对于程序员来说，超标量处理器对外体现的是顺序执行指令，其乱序执行和多发射的特性是硬件实现的，不需要程序员特定操作，就可以通过硬件优化程序性能；多核处理器则需要编写对应的多线程程序（例如借助OpenMP），才可以发挥多核处理器的性能。

## 5. 虚拟化
虚拟化的主要目的是为用户提供一个友好的使用界面，因为计算机硬件是复杂的，不好用的，虚拟化则为这个不好用的平台提供了一个好用的界面。虚拟化技术包括虚拟存储，虚拟机，流水线和多发射技术，cache技术，以及分布式共享存储系统中的cache一致性协议。

### 5.1 虚拟存储
操作系统提供虚拟空间，使得程序员可以不用直接和物理内存和外存打交道。操作系统自动完成虚拟地址到物理地址的转换以及数据在内存和外存之间的调入调出。虚拟存储虚拟了内存，具体可见操作系统的课程。

虚拟存储实现的一个重要功能是隔离，使用虚拟存储的概念，可以让计算机实现多进程的功能，不同的进程之间虚拟地址可能相同，但是通过地址转换映射到不同的物理地址，这样就保证了同时各个线程之间互不影响，进而实现多线程。

### 5.2 虚拟机
多线程技术，尤其是同时多线程（simultaneous multi-threading, SMT）技术，通过微结构的硬件支持，例如设立多组通用寄存器等，使得在同一个CPU中实现两个或多个线程中的指令在流水线中混合执行，或在同一个CPU中实现线程的快速切换，是用户可以“同时”在一个CPU上执行多个线程。虚拟机技术则通过微结构的硬件增强，如设立多组控制寄存器和系统状态等，实现多个操作系统的快速切换，达到在同一个计算机上“同时”运行多个操作系统的目的。

### 5.3 流水线和多发射结构
工艺的发展使得流水线和多发射结构得以实现，在维持串行编程模式的情况下提高了速度。

### 5.4 cache技术
使得CPU可以用到看上去和cache一样快，和内存一样大的存储空间，不用改应用程序就可以提升性能。但现代处理器往往80%以上的晶体管都用在了cache上，代价较大。


### 5.5 cache一致性协议
可以在分布式存储的情况下给程序员提供一个统一的编程空间，屏蔽了存储器物理分布的细节。但cache一致性协议没有解决程序员需要并行编程、原有的串行程序不能并行运行的问题。






## 参考
《计算机体系结构基础》 第三版 胡伟武 等著

《现代操作系统 原理与实现》 陈海波 夏虞斌 等著

《计算机原理与设计 Verilog HDL版》 李亚民 著

[超标量与多核的区别是什么](https://www.zhihu.com/question/295139731)