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

- 结构
  默克尔树是一种二叉树结构，其中每个非叶节点的值是其两个子节点值的哈希值的串联。最终的根节点哈希值代表整个数据集的完整性。

- 数据存储

  实际数据存储在叶节点上，而非树的中间节点。每个叶节点存储数据的哈希值，而不是数据本身。

- 数据哈希

  不同类型的数据可能需要使用不同的哈希方法，但这需要在使用默克尔树的各方之间进行协商和约定。

- 验证证明

  验证方需要校验某个叶节点的数据拥有权时，只需提供该叶节点的数据哈希值以及沿着路径上的所有中间节点的哈希值。通过重新计算这些哈希值，验证方可以验证数据的完整性和正确性。

  ![merkle tree]({{ "/assets/images/2024-01-28/merkle-tree.jpg" | absolute url }})

  例如上图,  证明方只需要提供黄色节点的源数据,以及所有红色的 H0 值, 校验方自己计算出所有绿色的 H0 值， 最后得到根 hash 值，与区块链上的根 hash 值进行比对即可。
