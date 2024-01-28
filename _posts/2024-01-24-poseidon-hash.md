---
layout: post
title: DID 学习日记 - 基础篇 - poseidon hash
subtitle: 一种 ZK 友好的 hash
categories: Web3 DID
tags: [DID, poseidon, hash]
---

## DID 学习日记 - 基础篇 - poseidon hash

### 官方资料

[https://www.poseidon-hash.info](https://www.poseidon-hash.info)

[Poseidon](https://eprint.iacr.org/2019/458.pdf)

[Poseidon2](https://eprint.iacr.org/2023/323.pdf)

### 我对 poseidon hash 的一些粗浅理解

#### 与其他 hash 共有的特点

poseidon hash 和其他 hash 函数一样， 都有如下特点:

- 抗碰撞性 (collision resistance)

  Poseidon 被设计为抗碰撞的哈希函数，这意味着在常规计算条件下，两个不同的输入应该产生不同的哈希输出。抗碰撞性是哈希函数的一项基本属性，确保不同的输入映射到相同的哈希值的概率非常小。

- 不可逆性 (pre-image resistance)

  不可逆性是指给定哈希输出，理论上很难找到原始的输入。在哈希函数的设计中，不可逆性是一个重要的安全属性，确保哈希值不能轻易被逆向计算出原始输入。

#### poseidon hash 的优势

POSEIDON hash 与其他 hash 函数在几个方面有所不同：

- 结构

  POSEIDON hash 采用了一种新颖的结构，包括了 Sponge Construction 和 HADES Design Strategy，这使得它在处理零知识证明系统中的特定需求时表现出色。

- 性能

  相比其他 hash 函数，POSEIDON hash 在特定场景下能够提供更高的性能，例如在 Merkle 树中累积值的零知识证明中，POSEIDON hash 可以在 1 秒内处理十亿大小的树，而且其约束数也比其他函数低。

- 应用

  POSEIDON hash 被建议用于各种零知识证明友好的哈希应用，包括承诺、签名方案和多元素对象的哈希等 7。

- 设计灵感

  POSEIDON hash 的设计灵感来自于多个密码学算法，如 LowMC 密码、SHARK 分组密码和 MiMC 算法，这使得其设计更加全面 4。

  这些方面的不同使得 POSEIDON hash 在特定的零知识证明系统中具有独特的优势，从而为密码学领域带来了新的可能性。
