---
layout: post
title: DID 学习日记 - 基础知识 - 默克尔树(merkle tree)
subtitle: 一种验证数据完整性的二叉树结构
categories: DID
tags: [DID, merkle-tree]
---

## DID 学习日记 - 基础知识 - 默克尔树(merkle tree)

### 资料

[wikipedia](https://en.wikipedia.org/wiki/Merkle_tree)

[Paper](https://people.eecs.berkeley.edu/~raluca/cs261-f15/readings/merkleodb.pdf)

### 我对默克尔树的一些粗浅理解

- 默克尔树是一种二叉树结构，每个节点都是其(两个)子节点的 hash 值，最终的根节点就是整个树的 hash 值。

- 默克尔树的数据信息是存储在叶节点上的，而非树的中间节点。

- 叶节点存放的也只是数据的 hash 值，而非数据本身。（不同类型的数据的 hash 方法需要使用该默克尔树的各方约定好）

- 证明方需要校验自己对默克尔数的某个叶节点的数据的拥有权，只需要提供该叶节点的数据的 hash 值，以及该叶节点的路径上的所有中间节点的 hash 值即可。

  ![merkle tree]({{ "/assets/images/2024-01-28/merkle-tree.jpg" | absolute url }})

  例如上图,  证明方只需要提供黄色节点的源数据,以及所有红色的 H0 值, 校验方自己计算出所有绿色的 H0 值， 最后得到根 hash 值，与区块链上的根 hash 值进行比对即可。
