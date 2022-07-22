---
layout: post
title: Verilog编程-12. 乘法器设计
categories: Verilog
description: 使用Verilog实现多种方式的有符号数乘法
keywords: Verilog
---

# Verilog编程-12. 乘法器设计

## 简介
传统的乘法器设计是采用移位相加的方法。而在Verilog中最简便的方式是使用`*`符号，这样DC在综合的时候会自动进行优化。但是这种设计并不具有灵活性，且无法采取更多种的优化手段。但是自己的设计在DC之后，可能还不如直接使用`*`符号，那么就说明自己的设计还是不如DC的优化设计，但是掌握原理进行训练仍然是必要的。DC应该是调用Designware的库，应该是优化力度最大的，所以一般比自己设计的面积和时序都好一些。

## 背景
### 阵列乘法

### Wallace树
Wallace树是一种树形加法结构，根据Wallace树实现的乘法器，改进了部分积累加结构，进而提升了乘法器性能。在Wallace树中，如果采用传统的行波进位方式，当前bit相加结果依赖于前一bit加法的进位输出，整个计算过程延时变大，所以优化的关键是消除进位链，使运算并行化，而使用进位保留加法器（CSA）是一种常用的手段，所以CSA是实现Wallace树的关键所在。

### Booth编码

Booth乘法器和Wallace树乘法器是两种不同的思路，分别可以实现乘法，两者也可以结合在一起使用，结合两者的优点，提升乘法器的性能。优化乘法器的方法中，booth算法和wallace树最为典型，booth算法可以减少部分乘积项，减少的是乘积项为0的部分，Wallace树可以提高部分乘积相加的并行性。在工程中常用的是MBE(Modified Booth Encoding)，便于重编码。

Booth乘法器是将乘数进行Booth编码之后再相乘的乘法器，是运用了Booth编码方式的乘法器。Booth编码之所以称之为编码，是因为其对原始的二进制进行了重编码，通常有基-2编码和基-4编码。原理如下所示：

Booth乘法器最大的特点是减少了乘积项，现实中常用的是基-4 Booth编码（Radix-4 Booth Encoding）。采用基-4编码的乘法相较于传统乘法运算，优化效果已经很明显了，且易于实现，可以满足大部分应用要求。当然更高阶的Booth编码可以更大程度地减少部分积个数，但其部分积产生逻辑已经无法单纯通过移位实现，需要引入加法器等其他运算部件，从这方面看则削弱了优化效果，需要综合考量选择，但其实现原理都是类似的。

## 设计思路
### 无符号数乘法器
关于无符号数乘法器的基本设计，可以参见这篇文章[Verilog编程-2](https://polaris-chn.github.io/2022/04/10/Verilog-practice-2/)

### 有符号数乘法器

### 阵列乘法器

### Wallace树乘法器

### Booth乘法器

### Booth编码+Wallace树乘法器


## 代码

## 仿真结果

## 参考
[基于Wallace树的4位乘法器实现](https://www.codeleading.com/article/72174189008/)

[Booth算法与Wallace树](https://www.wenhui.space/docs/08-ic-design/typical/booth-and-wallace/)

[图解Wallace树](https://zhuanlan.zhihu.com/p/130968045)

[进位保存加法器原理与设计](https://zhuanlan.zhihu.com/p/102387648)

[Booth原理与Verilog实现](https://zhuanlan.zhihu.com/p/127164011)

[改进的Booth乘法(基4)](https://www.cnblogs.com/lyc-seu/p/12890155.html)
