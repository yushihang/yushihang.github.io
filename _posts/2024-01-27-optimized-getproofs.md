---
layout: post
title: DID 学习日记 - PolygonID - 优化 getproofs 性能
subtitle: isolate + openmp + parallel
categories: Web3 DID
tags: [DID, PolygonID, openmp]
---

## DID 学习日记 - PolygonID - 优化 getproofs 性能

### 优化方案

通过日志打印确认了耗时的函数调用

总共进行了三点优化

1. 将原来串行的 for-loop + await, 改为 Future.await(), 即将每次 for-loop 迭代中的函数，在同一个 Future.await()中"并行进行"
2. 去掉了多余的 fetchSchema 调用， 原来的实现对每个 request 进行了两次 fetchSchema 调用，现在只进行一次。
3. 对 fastsnark native lib 在 iOS 上增加了 openmp 支持。

### 优化前的 sequence diagram:

(背景为红色的部分是相对耗时的函数)

![优化前的GetProofs]({{ "/assets/images/2024-01-27/optimize-getproofs.jpg" | absolute url }})

### 优化后的 sequence diagram:

(背景为红色的部分是相对耗时的函数)

![优化后的GetProofs]({{ "/assets/images/2024-01-27/after-optimized-getproofs.jpg" | absolute url }})
