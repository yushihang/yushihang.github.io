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

#### simplify.sym

```
1,1,1,main.out
2,2,1,main.in[0]
3,3,1,main.in[1]
4,-1,0,main.c.out
5,-1,0,main.c.in[0]
6,-1,0,main.c.in[1]
```

参照<https://docs.circom.io/circom-language/formats/sym/>，我们可以得到如下表格

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

  "6": { "0": "1", "2": "2", "3": "1" }对应的含义是, main.c.in[1] 在编译优化过程中被替换为了 1 \* 1 + main.in[0] \* 2 + main.in[1] \* 1

总结一下

|  原 Signal   |          替换后的表达式          |
| :----------: | :------------------------------: |
|  main.c.out  |             main.out             |
| main.c.in[0] |            main.in[0]            |
| main.c.in[1] | 1 + 2 \* main.in[0] + main.in[1] |

这和 circom 中的语义以及优化后的 r1cs.json 可以对应上

### example2

#### circuit code

```circuit

pragma circom 2.0.0;

template calc () {

   // Declaration of signals.
   signal T2;
   signal T1;
   signal z;

   signal output y;

   signal input x;

   //y = x ** 3 + 2x + 5
   // Constraints.
   T1 <== x * x;
   z <== T1 * x;
   T2 <== z + 2 * x;
   y <== T2 + 5;
}

component main  = calc();
```

#### compile

```console
❯ circom circuit.circom --r1cs --wasm --simplification_substitution --O2 --sym --inspect
template instances: 1
non-linear constraints: 2
linear constraints: 0
public inputs: 0
private inputs: 1
public outputs: 1
wires: 4
labels: 6
Written successfully: ./circuit.r1cs
Written successfully: ./circuit.sym
Written successfully: ./circuit_js/circuit.wasm
Everything went okay
```

#### export r1cs.json

```console
❯ snarkjs r1cs export json circuit.r1cs circuit.r1cs.json
[INFO]  snarkJS: undefined: Loading constraints: 0/2
[INFO]  snarkJS: undefined: Loading map: 0/4
```

#### circuit.sym

```
1,1,0,main.y
2,2,0,main.x
3,-1,0,main.T2
4,3,0,main.T1
5,-1,0,main.z
```

参照<https://docs.circom.io/circom-language/formats/sym/>，我们可以得到如下表格

| #s  | #w  | #c  | name |
| :-: | :-: | :-: | :--: |
|  1  |  1  |  0  |  y   |
|  2  |  2  |  0  |  x   |
|  3  | -1  |  0  |  T2  |
|  4  |  3  |  0  |  T1  |
|  5  | -1  |  0  |  z   |

编译后的 witness 数组为[ 1, y, x, T1 ]

#### r1cs

r1cs.json

```json
{
  "n8": 32,
  "prime": "21888242871839275222246405745257275088548364400416034343698204186575808495617",
  "nVars": 4,
  "nOutputs": 1,
  "nPubInputs": 0,
  "nPrvInputs": 1,
  "nLabels": 6,
  "nConstraints": 2,
  "useCustomGates": false,
  "constraints": [
    [
      {
        "2": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
      },
      {
        "2": "1"
      },
      {
        "3": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
      }
    ],
    [
      {
        "3": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
      },
      {
        "2": "1"
      },
      {
        "0": "5",
        "1": "21888242871839275222246405745257275088548364400416034343698204186575808495616",
        "2": "2"
      }
    ]
  ],
  "map": [0, 1, 2, 4],
  "customGates": [],
  "customGatesUses": []
}
```

#### w◯A \* w◯B = w◯C

| witness |  1  |  y  |  x  | T1  |
| :-----: | :-: | :-: | :-: | :-: |
|    A    |  0  |  0  | -1  |  0  |
|    B    |  3  |  0  |  1  |  0  |
|    C    |  0  |  0  |  0  | -1  |

w◯A 对应的表达式为 -x

w◯B 对应的表达式为 x

w◯C 对应的表达式为 -T1

w◯A \* w◯B = w◯C 也就是 `-x * x = -T1`

这对应着 circom 中的`T1 <== x * x;`

| witness |  1  |  y  |  x  | T1  |
| :-----: | :-: | :-: | :-: | :-: |
|    A    |  0  |  0  |  0  | -1  |
|    B    |  3  |  0  |  1  |  0  |
|    C    |  5  | -1  |  2  |  0  |

w◯A 对应的表达式为 -T1

w◯B 对应的表达式为 x

w◯C 对应的表达式为 5 - y + 2x

w◯A \* w◯B = w◯C 也就是 `-T1 * x = 5 - y + 2x`

等式两边移项

`y = T1 * x + 2x + 5`

对应着 circom 中如下代码优化后的结果

```circom
   z <== T1 * x;
   T2 <== z + 2 * x;
   y <== T2 + 5;
```

#### simplify_substitutions.json

```json
{
  "substitution": {
    "5": {
      "0": "21888242871839275222246405745257275088548364400416034343698204186575808495612",
      "1": "1",
      "2": "21888242871839275222246405745257275088548364400416034343698204186575808495615"
    },
    "3": {
      "0": "21888242871839275222246405745257275088548364400416034343698204186575808495612",
      "1": "1"
    }
  }
}
```

因为 r1cs 中 prime 值如下
`"prime": "21888242871839275222246405745257275088548364400416034343698204186575808495617"`

可以将 json 理解为如下内容

```json
{
  "substitution": {
    //z: (5,-1,0,main.z)
    "5": {
      "0": "-5", // -5 * 1
      "1": "1", //1 * y (1,1,0,main.y)
      "2": "-2" //-2 * x (2,2,0,main.x)
    },
    //T2: (3,-1,0,main.T2)
    "3": {
      "0": "-5", // -5 * 1
      "1": "1" //1 * y (1,1,0,main.y)
    }
  }
}
```

- "5": { "0": "-5", "1": "1", "2": "-2" }
  z 被优化为 -5 + y - 2x
  这与 circom 中的如下代码语义一致

  ```
  T2 <== z + 2 * x;
  y <== T2 + 5;
  ```

  代入消除 T2 后 `y = z + 2x + 5`

  移项后 `z = -5 + y - 2x`

- "3": { "0": "-5", "1": "1"}
  T2 被优化为 -5 + y
  这与 circom 中的如下代码语义一致

  ```
  y <== T2 + 5;
  ```

### example3

#### circuit code

```circuit
pragma circom 2.0.0;
template LinearAdder() {
    signal input a;
    signal input b;
    signal output sum;
    signal temp;

    temp <== a + b;
    sum <== temp * 2;
}

component main{public[a]} = LinearAdder();
```

#### compile

```console
❯ circom LinearAdder.circom --r1cs --wasm --simplification_substitution --O2 --sym --inspect

template instances: 1
non-linear constraints: 0
linear constraints: 0
public inputs: 1
private inputs: 1 (none belong to witness)
public outputs: 1
wires: 3
labels: 5
Written successfully: ./LinearAdder.r1cs
Written successfully: ./LinearAdder.sym
Written successfully: ./LinearAdder_js/LinearAdder.wasm
Everything went okay
❯ snarkjs r1cs export json LinearAdder.r1cs LinearAdder.r1cs.json
[INFO]  snarkJS: undefined: Loading map: 0/3
```

从结果看, 约束数量为 0

#### circuit.sym

```
1,1,0,main.sum
2,2,0,main.a
3,-1,0,main.b
4,-1,0,main.temp
```

编译后的 witness 数组为[1, sum, a]

#### r1cs

r1cs.json

```json
{
  "n8": 32,
  "prime": "21888242871839275222246405745257275088548364400416034343698204186575808495617",
  "nVars": 3,
  "nOutputs": 1,
  "nPubInputs": 1,
  "nPrvInputs": 1,
  "nLabels": 5,
  "nConstraints": 0,
  "useCustomGates": false,
  "constraints": [],
  "map": [0, 1, 2],
  "customGates": [],
  "customGatesUses": []
}
```

同样没有约束

#### simplify_substitutions.json

```json
{
  "substitution": {
    "3": {
      "0": "0",
      "1": "10944121435919637611123202872628637544274182200208017171849102093287904247809",
      "2": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
    },
    "4": {
      "0": "0",
      "1": "10944121435919637611123202872628637544274182200208017171849102093287904247809",
      "2": "0"
    }
  }
}
```

因为 r1cs 中 prime 值如下
`"prime": "21888242871839275222246405745257275088548364400416034343698204186575808495617"`

所以`21888242871839275222246405745257275088548364400416034343698204186575808495616`可以理解为`-1`

`10944121435919637611123202872628637544274182200208017171849102093287904247809`可以理解为`0.5`

可以将 json 理解为如下内容

```json
{
  "substitution": {
    // b: (3,-1,0,main.b)
    "3": {
      "0": "0", // 0 * 1
      "1": "0.5", // 0.5 * sum (1,1,0,main.sum)
      "2": "-1" //-1 * a (2,2,0,main.a)
    },
    // temp: (4,-1,0,main.temp)
    "4": {
      "0": "0", // 0 * 1
      "1": "0.5", // 0.5 * sum (1,1,0,main.sum)
      "2": "0" //0 * a (2,2,0,main.a)
    }
  }
}
```

这里其实已经跟<https://docs.circom.io/circom-language/formats/simplification-json/>中的`where the linear expression is represented by a dictionary with the signal numbers as strings occurring in the linear expression (with non-zero coefficient) as entries and their coefficients (as string) as values: { "sig_num_l1": "coef_1", ... , "sig_num_lm": "coef_m"}` 有冲突了

`"0": "0"` 的第二个"0"已经不是 with non-zero coefficient 了。

- "3": { "0": "0", "1": "0.5", "2": "-1" }

  `b` 被优化为 `0.5 * sum - a`

  转换后可以得到`sum = 2 * (a + b)`, 这跟 circom 的语义一致

- "4": { "0": "0", "1": "0.5", "2": "0" }
  `temp` 被优化为 `0.5 * sum`
  这与 circom 中的`sum <== temp * 2;` 语义一致
