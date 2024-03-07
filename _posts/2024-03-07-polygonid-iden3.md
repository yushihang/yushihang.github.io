---
layout: post
title: DID 学习日记 - PolygonID 和 Iden3 的关系和区别
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3, Iden3]
---

## DID 学习日记 - PolygonID 和 Iden3 的关系和区别

- 从 github 上的人员来看，这两个 group 的人员是有重复的

- PolygonID 的 Flutter SDK 实现 以及电路都是参照了 Iden3 的协议和电路代码

- PolygonID 在实现上还是进行了改造， 例如电路打包等

- 架构上的改动至少有两点

  - 客户段的 authclaim 不上链， 所以 private key 和 DID 有唯一确定关系， 也就是说客户端不能自己修改 key-pair 了
    而且客户端的三棵树一开始创建后就不变了

  - MTP VC 的实现方式有出入，是往一个固定的 slot 插入， 而不是 Iden3 文档说的
    <https://docs.iden3.io/getting-started/issue-claim-overview/#similarities-and-differences>

    polygonid 的文档压根没提这个事
    <https://devs.polygonid.com/docs/issuer/cred-issue-methods/#sig-method-issuance-of-credentials-with-baby-jubjubbjj-key-signatures>
