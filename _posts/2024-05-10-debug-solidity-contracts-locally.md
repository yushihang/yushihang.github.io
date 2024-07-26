---
layout: post
title: DID 学习日记 - 如何在本地调试智能合约
subtitle:
categories: DID Web3 SmartContracts
tags: [DID, Web3, SmartContracts, Ganache, hardhat, truffle]
---

## DID 学习日记 - 如何在本地调试智能合约

### 安装 Ganache

从<https://archive.trufflesuite.com/ganache/>下载 Ganache 安装包，运行后的图片如下:

![Ganache运行效果]({{ "/assets/images/2024-05-10/ganache.jpg" | absolute url }})

### 修改 Ganache 配置

点击右上角齿轮图标

![修改配置]({{ "/assets/images/2024-05-10/ganache-1.jpg" | absolute url }})

按需修改端口
![修改端口]({{ "/assets/images/2024-05-10/ganache-2.jpg" | absolute url }})

修改后点击右上角 Restart 按钮

### 下载 Iden3 智能合约代码

```bash
git clone https://github.com/iden3/contracts
```

### 安装 Node.js 相关工具/环境

```bash
cd contracts
npm install
npm install --save-dev hardhat@^2.19.0 @nomicfoundation/hardhat-toolbox@^3.0.0
npm instal -g pnpm
npm install ethers
pnpm init
pnpm install hardhat
pnpm exec hardhat # install stuf
pnpm exec hardhat compile
```

### 修改 hardhat.config.ts 文件

新增 `ganache` 配置

```typescript
const config: HardhatUserConfig = {

  ...

  networks: {

  ...

    ganache: {
      url: `http://127.0.0.1:9545`,
      accounts: [`0x5c3de83944f2df847a773cxxxxxxx2f74417068b02a5e15c7f72d0f3b6ef8f7`],
    },
  }
```

其中 url 中的端口是刚才我们按需修改的

accounts 的私钥如下图指引获取:
![获取私钥]({{ "/assets/images/2024-05-10/ganache-3.jpg" | absolute url }})

### 部署智能合约到 ganache

```bash
❯ npx hardhat run --network ganache scripts/deployState.ts
[ '======== State: deploy started ========' ]
[ 'found defaultIdType 0x1691 for chainId 1337' ]
[ 'deploying verifier...' ]
[
  'VerifierStateTransition contract deployed to address 0x648E72e6a2DF623Dd19036D40943A1318a275305 from 0xb065B82050E5EF5B7c414f3A32813D89319444CA'
]
[ 'deploying poseidons...' ]
Poseidon1Elements deployed to: 0x81AeA5D1941f5E1aDF463C03ec2f8523D07E4945
Poseidon2Elements deployed to: 0x76bab22056C8F5ab668F69eFa3E68703cA34636a
Poseidon3Elements deployed to: 0xBB6b0fD61281365e6AA57F74F21e5ccc1818C394
Poseidon4Elements deployed to: 0xe42ad74ADa823c28B509f889D04e5eeb46eE56a4
[ 'deploying SmtLib...' ]
[ 'SmtLib deployed to:  0x020f448791ECC886a501DF287F1610Cc6f2DB621' ]
[ 'deploying StateLib...' ]
[ 'StateLib deployed to:  0x47E6195FDe9D1B091F7683e30f983D13a742d4dF' ]
[ 'deploying state...' ]
Warning: Potentially unsafe deployment of contracts/state/State.sol:State

    You are using the `unsafeAllow.external-library-linking` flag to include external libraries.
    Make sure you have manually checked that the linked libraries are upgrade safe.

[
  'State contract deployed to address 0xe8AdCB79731b961b5aAF343857D5f9c9967913BD from 0xb065B82050E5EF5B7c414f3A32813D89319444CA'
]
[ '======== State: deploy completed ========' ]

```

### 运行测试用例， 测试部署后的合约函数调用

```bash
npx hardhat test --network ganache
```

### 运行指定用例

```bash
npx hardhat test --grep "Should be correct historical proof by root and the latest root";
```

grep 后的内容可以用正则表达式进行模糊匹配
对应的字符串为

```typescript
  it("Should be correct historical proof by root and the latest root", async function () {

    ...

    }
```

执行结果如下

```bash
npx hardhat test --grep "Should be correct historical proof by root and the latest root";

GenesisUtilsWrapper deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
PrimitiveUtilsWrapper deployed to: 0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
  GIST proofs
Poseidon1Elements deployed to: 0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9
Poseidon2Elements deployed to: 0xDc64a140Aa3E981100a9becA4E685f962f0cF6C9
Poseidon3Elements deployed to: 0x5FC8d32690cc91D4c39d9d3abcBD16989F875707
Poseidon4Elements deployed to: 0x0165878A594ca255338adfa4d48449f69242Eb8F
Warning: Potentially unsafe deployment of contracts/state/State.sol:State

    You are using the `unsafeAllow.external-library-linking` flag to include external libraries.
    Make sure you have manually checked that the linked libraries are upgrade safe.

============= getProofByRoot =============
=== depth:  0
node type: 1
node childLeft: 0
node childRight: 0
node index: 10383224237730897866344331752565975881613706181434347887058938829322673753863
node value: 2199023255552
getGISTProof =  Result(8) [
  18217902562915912830381625214985336486256409405650847383872545192881751549308n,
  false,
  Result(64) [
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n
  ],
  714108189666787013475035030611180720857198612789007010464764692850146621785n,
  2199023255552n,
  true,
  10383224237730897866344331752565975881613706181434347887058938829322673753863n,
  2199023255552n
]
============= getProofByRoot =============
=== depth:  0
node type: 1
node childLeft: 0
node childRight: 0
node index: 10383224237730897866344331752565975881613706181434347887058938829322673753863
node value: 2199023255552
lastProofRoot =  18217902562915912830381625214985336486256409405650847383872545192881751549308n
============= getProofByRoot =============
=== depth:  0
node type: 1
node childLeft: 0
node childRight: 0
node index: 10383224237730897866344331752565975881613706181434347887058938829322673753863
node value: 3298534883328
getGISTProof =  Result(8) [
  21869641454058577676054823291165674824712941839556468464464828229021598069978n,
  false,
  Result(64) [
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n,
    0n, 0n, 0n, 0n
  ],
  714108189666787013475035030611180720857198612789007010464764692850146621785n,
  3298534883328n,
  true,
  10383224237730897866344331752565975881613706181434347887058938829322673753863n,
  3298534883328n
]
============= getProofByRoot =============
=== depth:  0
node type: 1
node childLeft: 0
node childRight: 0
node index: 10383224237730897866344331752565975881613706181434347887058938829322673753863
node value: 3298534883328
lastProofRoot =  21869641454058577676054823291165674824712941839556468464464828229021598069978n
============= getProofByRoot =============
=== depth:  0
node type: 1
node childLeft: 0
node childRight: 0
node index: 10383224237730897866344331752565975881613706181434347887058938829322673753863
node value: 2199023255552
============= getProofByRoot =============
=== depth:  0
node type: 1
node childLeft: 0
node childRight: 0
node index: 10383224237730897866344331752565975881613706181434347887058938829322673753863
node value: 3298534883328
    ✔ Should be correct historical proof by root and the latest root (321ms)


  1 passing (2s)
```

### 在合约代码中打印日志

```solidity

pragma solidity 0.8.20;

import "hardhat/console.sol"; //添加这段代码

...


console.log("============= getProofByRoot ============="); //需要打印日志的地方

```

### abi json

执行如下语句编译智能合约

```bash
npx hardhat compile
```

生成好的 abi.json 会位于 ./artifacts/contracts/state/State.sol/State.json

### 收获

我用在合约中调试的方法，搞明白了稀疏默克尔树中的 node_aux 的含义(合约代码比 js 和 go 都要清晰一些)

1. 当要根据 node 的 path 去稀疏默克尔树中寻找一个节点时， 如果根据 path 寻找左右子节点的过程中遇到了叶子节点 node_aux。
2. 而这个叶子节点的 index 和 node.index 不同，这就说明 node 不在这棵树里。(node 如果在树里，那么用 path 找到的叶子节点应该就是 node，有一样的 index)

3. 所以 node_aux 的 index 和 value 需要被作为 MTProof 的一部分和 siblings 数据一起提交， 证明这些数据可以计算出 root state。从而证明我们需要寻找的 node 真的不在树里

4. 是否可以用上一层的节点的 hash 来伪造 node_aux？假装自己不在树里？
   我觉得 sibling 的 hash 和 node 的 index 和 value 的数量级是不同的， 做不到伪造。
