---
layout: post
title: Verilog编程-12. 乘法器设计
categories: Verilog
description: 使用Verilog实现多种方式的有符号数乘法
keywords: Verilog
---

# Verilog编程-12. 乘法器设计

## 简介
传统的乘法器设计是采用移位相加的方法。而在Verilog中最简便的方式是使用`*`符号，这样DC在综合的时候会自动进行优化。

## 背景
### 阵列乘法

### Wallace树

### Booth编码

Booth乘法器和Wallace树乘法器是两种不同的思路，分别可以实现乘法，两者也可以结合在一起使用，结合两者的优点，提升乘法器的性能。优化乘法器的方法中，booth算法和wallace树最为典型，booth算法可以减少部分乘积项，减少的是乘积项为0的部分，Wallace树可以提高部分乘积相加的并行性。在工程中常用的是MBE(Modified Booth Encoding)，便于重编码。

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
