---
layout: post
title: Web3学习日记 - Wallet Connect 分析 (二)
subtitle:
categories: WalletConnect DApp BlockChain-Wallet
tags: [WalletConnect, BlockChain-Wallet, DApp, ECDHE]
---

## Web3 学习日记 - Wallet Connect 分析 (二)

### Wallet Connect SDK 分类

Wallet Connect 有两套 SDK

AppKit (以前叫 Web3Modal)

WalletKit (以前叫 Web3Wallet)

相关页面见: <https://docs.walletconnect.com>

AppKit 主要给 DApp 接入

WalletKit 主要给 Wallet 接入

### Wallet Connect 网络架构

![网络架构]({{ "/assets/images/2024-06-25/WalletConnect-network-arch.jpg" | absolute url }})

#### Wallet Connect 使用了什么作为 Relay Network?

从这篇文档里看: <https://walletconnect.com/blog/walletconnect-protocol-v20-whats-new>

WalletConnect v2.0 uses Waku 2.0 by default to relay messages in a decentralized manner.

看起来 Wallet Connect 使用的是 Waku

Waku 是一个基于 libp2p 的通信协议，用于在去中心化应用之间传递消息。

#### Waku 的相关知识

- 文档
  <https://docs.waku.org>
  <https://docs.waku.org/learn/waku-network/>
  <https://arxiv.org/pdf/2207.00038>

- Pub/Sub
  <https://docs.waku.org/learn/glossary#pubsub>
  <https://docs.waku.org/learn/glossary#pubsub-topic>
  订阅了某个 topic 的 peer 可以收到其他 peer 对这个 topic 发布的所有消息

- Store
  <https://docs.waku.org/learn/glossary#store>
  支持对消息的短时间存储，离线的 peer 也可以收到之前发送到 topic 上的消息

  需要注意的是, 在<https://docs.waku.org/learn/faq/>中提到了 Waku 和 ipfs 不同，并不能提供大量数据的长期存储，而且不能保证离线节点一定可以收到所有之前发送的消息。
  他们也建议使用<https://code.storage>来提供更稳定的存储

#### Waku 架构

![Waku]({{ "/assets/images/2024-06-25/waku-intro.jpg" | absolute url }})

#### 关于 WalletConnect 2.x 是否仍在使用 Waku 2.0 作为 Relay Network 的疑惑

从各种文档的时间线来看

- 2021-11-12

  <https://walletconnect.com/blog/walletconnect-protocol-v20-whats-new>

  "WalletConnect v2.0 uses Waku 2.0 by default to relay messages in a decentralized manner."

  这里开始提到 WalletConnect v2.0 使用 Waku 2.0

- 2022-06-30

  <https://arxiv.org/pdf/2207.00038>

  "At the time of this article, Waku is deployed by Status and WalletConnect v2."

  从这个描述看, WalletConnect 参与了 Waku 的部署落地。

- 2023-11-01

  <https://x.com/Waku_org/status/1719709627535741106>
  "WalletConnect runs a network similar to Waku that enables data exchange between devices (for example, DApps requesting transaction signatures to a wallet)."

  这篇推文的主要目的是说 WalletConnect 使用了部分中心化的方案，所以会因为地缘政治而受到压力，对某些地方的用户关闭服务。
  而 Waku 使用的是纯去中心化服务，也就无法实现这一点，反而没有这个压力问题。

  其中提到了"WalletConnect 使用的是和 Waku 类似的网络(而不是 Waku 本身)"

  这里已经跟 WalletConnect 的文档有矛盾了。

- 2024-05-17

  <https://specs.walletconnect.com/2.0/glossary/relay>
  "By default, the clients will use a proxy server connected to the Waku network and it will connect to clients through a WebSocket using the reference Relay API."

  但是在 2024 年 5 月更新的文档里，还是提到了 Waku Network.
  (当然这里也可以理解为是 WalletConnect 运营维护的 Waku 协议的网络， 而不是我们说的 Waku 社区运营的 relay 网络本身)

- 2024-06-24

  在 Waku Discord 频道里的询问， Waku 方面工作人员提到
  "Yep, WC tested Waku at the time but they pulled back to build their own network afaik"

  如果这个描述为真， 那么看起来就是 "WalletConnect 运营维护的 Waku 协议的网络， 而不是我们说的 Waku 社区运营的 relay 网络本身"

  当然这个有待于 WalletConnect 方面的确认。

  另外可以作为辅助确认的是: 在最新的 WalletConnect 源码里，pub / sub / topic / relay 这些 waku 上的概念都能找到对应的使用， 但是 protocol 不是 waku, 而是 irn。

  这个 irn 是代表什么暂时不清楚。

- Update 2024-07-07

  参见这里 <https://github.com/orgs/WalletConnect/discussions/4637>

  总结如下:

  1. 所有的 server 和 relay network 都是 walletconnect 来负担的， 他们没有提到收费计划

  2. walletconnect 虽然在很多文档里说他们的 Relay network 用了 Waku ，但是他们确认，曾经是使用过 Waku， 但是因为没有很好的解决延迟问题，现在已经用回了中心化的 Relay 网络 。

     他们文档里还提到使用了 Waku 的部分，是因为文档没有及时更新。

  3. 关于中心化的 relay network 是否可以自己部署的问题，我看 WallectConnect 讨论区里很多人也提出了类似问题。 他们的答复是说这个不在计划表里， 他们的主要计划是去中心化网络。

     我也再次问了一下他们，最新的计划是否仍然如此，他们说是的。

     我就解释了一下，如果不能自己部署 relay network 的话， 可能遇到内部安全审查问题，他给了一个他们 WallectConnect 找第三方（Trail of Bits）进行安全审查的报告链接，里面有审计报告原文和他们对审计结果的一些处理。
     也给了一个 WalletConect 负责安全审查的邮箱，说有疑问可以联系。
