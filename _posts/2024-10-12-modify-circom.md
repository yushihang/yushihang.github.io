---
layout: post
title: 一种可能的恶意修改 circom 的 ZKProof攻击方式
subtitle: 注意 assert 和 === 约束的使用
categories: Web3 zkproof snarkjs circom circuit
tags: [Web3, zkproof, snarkjs, circom, circuit]
---

## 一种可能的恶意修改 circom 的 ZKProof 攻击方式

### 流程图

![流程图]({{ "/assets/images/2024-10-12/iden3-assert-issue.jpg" | absolute url }})

### 防范建议

- 不要在 verification 依赖 assert() 进行检查, 因为 verification 阶段依赖的都是 circom 编译时导出的 r1cs 约束信息, 而 assert()只在编译阶段和 witness 生成阶段有效, 在 verification 阶段无效。

- 尽量使用 circomlib 中的库 template, 例如 <https://github.com/iden3/circomlib/blob/master/circuits/comparators.circom>中提供的内容, 在绝大多数情况下这个都是相对稳妥的方案。虽然在某些情况下他不是约束数量最少的解决方案。

- === 虽然可以提供约束, 但是在某些情况下会在编译过程被优化掉。所以需要做好约束检查，例如通过`snarkjs r1cs print circuit.r1cs circuit.sym` 命令进行检查。
