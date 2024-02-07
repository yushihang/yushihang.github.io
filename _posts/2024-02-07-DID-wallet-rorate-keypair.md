---
layout: post
title: DID 学习日记 - PolygonID - 修改钱包公私钥
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3]
---

## DID 学习日记 - PolygonID - 替换公私钥

在 client wallet 的 state 不上链的情况下，因为公钥通过各种 hash 和 authclaim + 三棵树唯一确定了 genesisDID
所以如果修改了私钥/公钥，公钥便无法和 genesisDID 对应， authV2 电路便无法验证 wallet 为这个 did 的持有者。

但是在 wallet 把 state 上链的情况下， 在 claim tree 里存在至少一个没失效的 operational key authorization 类型的 claims 的前提下， 可以用这个 operational key authorization claim 里的私钥对 message 签名，然后证明公钥和签名对应，同时又提供公钥和这个 claimtree 的某个叶子节点 hash 值对应（公钥存在于这个 tree 中），并能提供所有的 sibling value， 最后能计算出和链上一样的 state 值。

这样 publickey 和 genesisDID 就不是简单的 hash 关系了， 而是默克尔树的节点和根 state 的对应关系。

参考资料
<https://docs.iden3.io/protocol/spec/#identity-ownership> 中查找 Identity Key Rotation
