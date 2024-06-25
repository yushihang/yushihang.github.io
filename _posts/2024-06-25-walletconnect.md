---
layout: post
title: Web3学习日记 - Wallet Connect 分析 (一)
subtitle:
categories: WalletConnect DApp BlockChain-Wallet
tags: [WalletConnect, BlockChain-Wallet, DApp, ECDHE]
---

## Web3 学习日记 - Wallet Connect 分析

### Wallet Connect 介绍

Wallet Connect 是一个开源协议，用于连接去中心化应用程序（DApps）和钱包。它允许用户通过扫描二维码/粘贴 Url/唤醒 App，使用他们的移动钱包（如 MetaMask、Trust Wallet 等）来与 DApps 进行安全的交互，而无需在 DApp 中输入私钥或导入钱包。

DApp 通常不具有区块链钱包的功能， 所以用户在使用 DApp 时，一般需要链接到一个区块链钱包来实现和私钥相关的操作， 例如签署交易，签名数据等。

当然 DApp 也可以选择接入钱包 App 提供的 SDK 来完成，但是如果几百个区块链钱包， DApp 一家一家去接入的话，成本会很高。所以接入 Wallet Connect 这种公开协议是一个很好的选择。 一旦接入完成，理论上就可以和所有支持这个协议的钱包进行交互。

### 用例

#### 案例 1：DApp 连接钱包

- 用户在移动设备上打开去中心化交易所的移动应用（如 Uniswap 的移动端）。
- 用户选择通过 Wallet Connect 连接钱包。
- 应用生成一个 Wallet Connect 链接(WalletConnectURI)并拉起用户的钱包应用
- 用户在钱包 App 中确认连接到 DApp 的账户信息。
- 钱包应用与去中心化交易所的移动应用建立连接
- Wallet 自动跳转回 DApp 或者用户手动切换回 DApp
- 用户可以在 DApp 中查看账户地址，资产等信息。

#### 案例 2：去中心化交易所（DEX）的 DApp 链接钱包进行交易

场景：用户希望在去中心化交易所（DEX）上的移动应用进行交易。

- DApp 发起交易信息，生成一个交易请求。
- 通过 DeepLink 把 RPC 请求发送给连接后的钱包 App
- 用户在钱包应用中确认交易，交易在区块链上执行，结果返回到去中心化交易所的移动应用上。

#### 案例 3：签署需要在智能合约上执行的数据

场景：用户需要在去中心化应用（dApp）的移动应用上签署需要在智能合约上执行的数据。

- DApp 生成一个签名请求。
- 通过 DeepLink 把 RPC 请求发送给连接后的钱包 App
- 用户在钱包应用中可以看到需要签署的智能合约详情。
- 用户在钱包应用中确认并签署需要在智能合约上执行的数据，签名的交易被发送到区块链上。
- 签署完成后，DApp 的移动应用会更新状态，显示智能合约已成功签署。

### 对应的 RPC Method 列表

<https://docs.walletconnect.com/advanced/multichain/rpc-reference/ethereum-rpc#eth_signtypeddata>

### WalletConnect 视频

<embed src='{{ "/assets/video/2024-06-25/walletconnect.mp4"" | absolute url }}'>
