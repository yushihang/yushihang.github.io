---
layout: post
title: Circom 中赋值(<--)和约束(<==)的关系和区别
subtitle: 如果把<==都换成<--, 会发生什么?
categories: Web3
tags: [Circom, zkproof]
---

## Circom 中赋值(<--)和约束(<==)的关系和区别

### 我的疑惑

在学习 Circom 的时候，我发现 Circom 中有两种赋值的方式，一种是 `<--`，一种是 `<==`。

文档上提到，`<--` 是赋值，`<==` 是赋值+约束。

也就是说

```circom
a <== b
```

其实是如下代码的语法糖

```circom
a <-- b
a === b
```

但是如果以其他编程语言的经验来理解，将 A 赋值给 B 之后， A 和 B 的值肯定相等。

所以为什么会有 `<==` 和 `==>` 这两种操作符呢？

### 目前的理解

因为目前还在对 Circom 的机制深入学习中，不敢说自己一定理解了这种设计的哲学, 也不确认自己的理解都是正确的。

只能罗列一下可能对这个疑问有帮助的理解，待后续补充。

- `<==` 和 `==>` 只能对 Signal 进行操作， 不能对普通的变量进行操作

- Circom 的电路操作跟其他编程语言的顺序执行不同, 我的理解是他只是把各种 Signal "连通"在一起，并不强调先后顺序。最后电路"启动"后，各个 Signal 会按照拓扑排序的顺序进行计算。

- 同一个 Signal 不能被两次赋值 (连接)
  如下代码在编译时会报错

  ```circom
    template AgeProof() {
    signal input age;
    signal input ageLimit;
    signal output out;

    component lt = LessThan(32);
    lt.in[0] <== ageLimit;
    lt.in[1] <== age;
    out <== lt.out;

    out <== ageLimit; //this will cause error
  }
  ```

  ```bash
  ❯ circom AgeProof.circom --r1cs --wasm --sym --c
  error[T3001]: Exception caused by invalid assignment: signal already assigned
    ┌─ "AgeProof.circom":141:5
    │
  141 │     out <== ageLimit;
    │     ^^^^^^^^^^^^^^^^ found here
    │
    = call trace:
      ->AgeProof

  previous errors were found
  ```

  报错具体信息为`signal already assigned`

- Circom 的赋值运算和约束检查是两个不同的阶段
  赋值运算的逻辑是由 prover 在生成 witness 的过程中确定的，而约束检查是由 verifier 在验证 proof 的过程中进行的。

所以如果遇到恶意的 prover, 他是可以修改代码再进行赋值的。

如果约束的内容足够多，verifier 会在验证 proof 的时候发现这个 proof 是错误的。

但是如果 verifier 处运行的 circom 代码(不会被 proof 修改), 没有足够的约束， 例如都是 `<--` 而不是 `<==`, 就不一定能发现恶意修改代码后生成的 witness.

### 一些有用的文档

- [Assigned but not Constrained](https://github.com/0xPARC/zk-bug-tracker#8-assigned-but-not-constrained)

这个文档中用了一个很好的例子来说明 `<==` 和 `<--` 的区别。

也就是如果应该用 `<==` 的地方用了 `<--`，会导致的问题。

- [Execution vs constraints](https://dev.to/spalladino/a-beginners-intro-to-coding-zero-knowledge-proofs-c56#:%7E:text=For%20instance%2C%20Circom%20only%20allows,be%20the%20set%20of%20constraints)

为防止原文链接失效，我把对应的 Section 摘录在这里

#### Execution vs constraints

In the workflow above, you may have noted that the circuit is used twice by the prover: first to generate the witness, and then to derive the constraints that are covered by the proof. These two runs are completely different beasts, and understanding their difference is one of the keys to understanding how ZKPs work.

Let's go back to our Circom example from before, where we had two instructions:

```circom
ab <== a _ b;
c <== ab _ ab;
```

In Circom, a fat arrow is just syntax sugar for two different instructions, assignment and constrain, so the above can be expanded to:

```circom
ab <-- a*b
ab === a*b
c <-- ab _ ab
c === ab _ ab
```

The a <-- a*b instructions means during the execution phase, assign a*b to ab, and gets ignored when compiling the constraints. On the other hand, ab === a*b means add a constraint that forces ab to be equal to a*b, which gets ignored during execution. In other words, when writing a circuit you're writing two different programs, that belong to two different programming paradigms, in a single one.

While you will usually write assignments and constraints that are equivalent, sometimes you need to split them up. A good example of this is the IsZero circuit.

- [Constraint Generation](https://docs.circom.io/circom-language/constraint-generation/)

  Circom 文档中对`生成约束`的描述
