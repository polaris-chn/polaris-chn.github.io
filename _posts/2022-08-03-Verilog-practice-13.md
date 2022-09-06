---
layout: post
title: Verilog编程-13. 除法器设计
categories: Verilog
description: 使用Verilog实现多种方式的有符号数除法
keywords: Verilog
---

# Verilog编程-13. 除法器设计

## 简介

### 恢复余数除法器
采用恢复余数法（Restoring Division Algorithm, RDA）设计除法器，RDA无法直接用于有符号数，对于有符号数需要先转换为无符号数，然后根据除数与被除数的符号来判断商和余数的符号。

### 不恢复余数除法器
采用不恢复余数法（Non-Restoring Division Algorithm, NRDA）设计除法器。在恢复余数除法算法中，如果部分余数为负，则要恢复原来的余数并左移。不恢复余数的思想就是，不管相减结果是正是负，都把它写入到reg_r，若为负，下次迭代不是从中减去除数而是加上除数。


### SRT除法器
采用SRT算法设计除法器，SRT算法是以D.Sweeney, J.E.Robertson和T.D.Tocher三位科学家的姓氏首字母组合命名的算法，旨在加速非恢复二进制除法。


## 参考
[恢复余数除法器](https://zhuanlan.zhihu.com/p/164633088)

[不恢复余数除法器](https://zhuanlan.zhihu.com/p/206770701)

