---
layout: post
title: 计算机体系拓扑基础
categories: CA
description: 多核心处理器连接的拓扑结构的基础
keywords: CA, Topology, NoC, Ring, Mesh
---

# 1. 拓扑基础
在算力芯片中，芯片拓扑（Topology）结构非常重要，仅靠单核心CPU很难做出算力强且并行度高的芯片，如果拓扑结构设计得不合理，非常影响多芯片整合在一起的工作效率，很容易达到节点
上限，所以业界重点从单核心处理器微架构过渡到多核心拓扑结构。
