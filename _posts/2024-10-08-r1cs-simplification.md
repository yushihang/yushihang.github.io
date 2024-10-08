---
layout: post
title: 解析circom编译过程中的.sym和substitutions.json文件
subtitle: circom circuit.circom --r1cs --wasm --simplification_substitution --O2 --sym
categories: Web3 zkproof snarkjs circom circuit
tags: [Web3, zkproof, snarkjs, circom, circuit]
---

## 解析 circom 编译过程中的.sym 和 substitutions.json 文件

### circom 文档

<https://docs.circom.io/circom-language/formats/sym/>

<https://docs.circom.io/circom-language/formats/constraints-json/>

<https://docs.circom.io/circom-language/formats/simplification-json/>

### example1

#### circuit code

```circuit
pragma circom 2.0.0;

template Internal() {
   signal input in[2];
   signal output out;
   out <== in[0]*in[1];
}

template Main1() {
   signal input in[2];
   signal output out;
   component c = Internal ();
   c.in[0] <== in[0];
   c.in[1] <== in[1]+2*in[0]+1;
   c.out ==> out;
}


component main = Main1();
```

#### compile

```console
❯ circom simplify.circom --r1cs --wasm --simplification_substitution --O2 --sym --inspect
template instances: 2
non-linear constraints: 1
linear constraints: 0
public inputs: 0
private inputs: 2
public outputs: 1
wires: 4
labels: 7
Written successfully: ./simplify.r1cs
Written successfully: ./simplify.sym
Written successfully: ./simplify_js/simplify.wasm
Everything went okay
```

注意其中的--simplification_substitution --O2 参数

#### export r1cs.json

```console
❯ snarkjs r1cs export json simplify.r1cs simplify.r1cs.json
[INFO]  snarkJS: undefined: Loading constraints: 0/1
[INFO]  snarkJS: undefined: Loading map: 0/4
```

#### simplfy.sym

```
1,1,1,main.out
2,2,1,main.in[0]
3,3,1,main.in[1]
4,-1,0,main.c.out
5,-1,0,main.c.in[0]
6,-1,0,main.c.in[1]
```

根据<https://docs.circom.io/circom-language/formats/sym/>中的解释，我们可以得到如下的表格

| #s  | #w  | #c  |     name     |
| :-: | :-: | :-: | :----------: |
|  1  |  1  |  1  |   main.out   |
|  2  |  2  |  1  |  main.in[0]  |
|  3  |  3  |  1  |  main.in[1]  |
|  4  | -1  |  0  |  main.c.out  |
|  5  | -1  |  0  | main.c.in[0] |
|  6  | -1  |  0  | main.c.in[1] |

- #c 表示 signal 所属的 component

  可以类比为其他编程语言里的变量所属的函数。

  因为我们的 circom 代码里有两个 component, 所以可以看到这六个 signal 各有三个属于这两个 component。

- #s 是 signal 的编号，从 1 开始

- #w 是 signal 在 witness 数组中的 index

  如果使用 --O0 指定不优化, 那么一般来说 #w 和 #s 的值是一样的

  因为我们这个例子使用了 --O2, 所以我们可以看到 #s 为 4~6 的三个 signal 其实被优化了, 他们的#w 为-1

  换句话说, 这个 circom 编译后的 witness 数组为[ 1, main.out, main.in[0], main.in[1] ]

#### r1cs

r1cs.json

```json
{
  "n8": 32,
  "prime": "21888242871839275222246405745257275088548364400416034343698204186575808495617",
  "nVars": 4,
  "nOutputs": 1,
  "nPubInputs": 0,
  "nPrvInputs": 2,
  "nLabels": 7,
  "nConstraints": 1,
  "useCustomGates": false,
  "constraints": [
    [
      {
        "2": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
      },
      {
        "0": "1",
        "2": "2",
        "3": "1"
      },
      {
        "1": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
      }
    ]
  ],
  "map": [0, 1, 2, 3],
  "customGates": [],
  "customGatesUses": []
}
```

#### w◯A \* w◯B = w◯C

| witness |  1  | main.out | main.in[0] | main.in[1] |
| :-----: | :-: | :------: | :--------: | :--------: |
|    A    |  0  |    0     |     -1     |     0      |
|    B    |  1  |    0     |     2      |     1      |
|    C    |  0  |    -1    |     0      |     0      |

- w◯A: -main.in[0]
- w◯B: 1 + 2\*main.in[0] + main.in[1]
- w◯C: -main.c.out

- w◯A \* w◯B = w◯C

  (-main.in[0]) \* (1 + 2\*main.in[0] + main.in[1]) = -main.c.out

  main.c.out = main.in[0] \* (1 + 2\*main.in[0] + main.in[1])

  与 circom 中的语义一致

#### simplify_substitutions.json

```json
{
  "substitution": {
    "5": { "2": "1" },
    "4": { "1": "1" },
    "6": { "0": "1", "2": "2", "3": "1" }
  }
}
```

- "5": { "2": "1" }

  5 代表.sym 中的 main.c.in[0] (5,-1,0,main.c.in[0])
  2 代表.sym 中的 main.in[0] (2,2,1,main.in[0])
  1 代表 main.in[0]的系数, 即 main.in[0] \* 1

  "5": { "2": "1" }对应的含义是, main.c.in[0]在编译优化过程中被替换为了 main.in[0] \* 1

- "4": { "1": "1" }

  4 代表.sym 中的 main.c.out (4,-1,0,main.c.out)
  "1": "1"的第一个 1 代表 main.out (1,1,1,main.out)
  "1": "1"的第二个 1 代表 main.out 的系数, 即 main.out \* 1

  "4": { "1": "1" }对应的含义是, main.c.out 在编译优化过程中被替换为了 main.out \* 1

- "6": { "0": "1", "2": "2", "3": "1" }

  6 代表.sym 中的 main.c.in[1] (6,-1,0,main.c.in[1])
  "0": "1" 代表 1 \* 1 (下标为 0 的 witness 是 1)
  "2": "2" 代表 main.in[0] \* 2 (下标为 2 的 witness 是 main.in[0]: 2,2,1,main.in[0])
  "3": "1" 代表 main.in[1] \* 1 (下标为 3 的 witness 是 main.in[1]: 3,3,1,main.in[1])

  "6": { "0": "1", "2": "2", "3": "1" }对应的含义是, main.c.in[1] 在编译优化过程中被替换为了 1 \* 1 + 1 \* main.in[0] \* 2 + main.in[1] \* 1

总结一下
| 原 Signal | 替换后的表达式|
| :-----: | :-: |
|main.c.out| main.out|
|main.c.in[0]| main.in[0]|
|main.c.in[1]| 1 + 2\*main.in[0] + main.in[1]|

这和 circom 中的语义以及优化后的 r1cs.json 可以对应上
