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
