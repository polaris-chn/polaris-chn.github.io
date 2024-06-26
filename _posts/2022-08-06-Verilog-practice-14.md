---
layout: post
title: Verilog编程-14. 浮点数处理
categories: Verilog
description: 使用Verilog实现多种方式的浮点数处理
keywords: Verilog
---

# Verilog编程-14. 浮点数处理


## 简介
## 原理

### 浮点数加减法原理
二进制浮点数只有在指数相同的时候，才能进行尾数（小数）的加减法。所以在进行浮点数加减法的时候，先将两个操作数的指数幂调至相等，然后再进行尾数的加减法。具体分为对阶，尾数加减和规格化处理三个步骤。需要注意的是，在对阶的时候尾数需要移位，移位时隐含的1也要包括进来。在加减的过程中为了防止结果有进位或借位溢出，需增加结果的位数。两个浮点数符号位相同时，尾数相加；不同时，尾数相减。

对阶的过程中规定小阶向大阶对阶，即小阶的尾数进行移位，尾数向右移动，每移动一位，指数加1，直到阶数相等即完成对阶

对新得到的指数和尾数进行规格化和舍入操作（指数和尾数都要进行规格化和舍入），获得新的指数和尾数


### 浮点数乘除法原理
浮点数乘法主要包括四个步骤，指数加法、尾数相乘、符号位计算、规格化。

## 参考
[浮点数处理](https://www.yuejianzun.xyz/2019/05/28/%E6%B5%AE%E7%82%B9%E6%95%B0%E5%A4%84%E7%90%86/)

[浮点运算器设计](https://zhuanlan.zhihu.com/p/356960443)