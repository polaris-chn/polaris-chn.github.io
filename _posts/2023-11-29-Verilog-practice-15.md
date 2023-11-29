---
layout: post
title: Verilog编程-15. 算术右移的问题
categories: Verilog
description: 使用Verilog实现算术右移的坑
keywords: Verilog
---

# Verilog编程-15. 算术右移的问题

在verilog中实现算术右移，使用`>>>`符号，例如`in >>> 5`，即代表向右算术右移5位。

但问题在于，此时verilog将`in`视为无符号数，而无符号数逻辑右移（`>>`）和算术右移没有区别，都是高位补0。此时可使用`$signed(in)`将`in`转变为有符号数，此时就会在高位补符号位。

`$signed`和`$unsigned`都是可综合的。`$signed`是将无符号数转变为有符号数返回，不改变数的类型和内容，`$unsigned`同理。
