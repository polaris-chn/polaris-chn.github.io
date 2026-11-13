---
layout: post
title: 分支预测
categories: CA
description: 分支预测基本概念、基本算法
keywords: branch predictor、PC、gshare、btb
---

# 分支预测

## 1. 基本概念

### 1.1 动态分支预测
动态分支预测是能够根据处理器实际的执行情况，动态地对分支指令进行预测；不会简单地预测分支指令一直发生跳转或者不跳转，而是会根据分支指令在过去一段时间的执行情况来决定预测的结果，也就是称之为“动态”的原因。