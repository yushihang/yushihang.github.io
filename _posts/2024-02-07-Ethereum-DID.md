---
layout: post
title: DID 学习日记 - PolygonID - 基于 Ethereum address 的DID
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3]
---

## DID 学习日记 - PolygonID - 基于 Ethereum address 的 DID

<https://docs.iden3.io/getting-started/identity/identity-types/#limitations-of-ethereum-controlled-identity>

里面提到的 JWS with Ethereum Signature 可以参考如下资料

<https://learnblockchain.cn/article/5012>

<https://mirror.xyz/0x9B5b7b8290c23dD619ceaC1ebcCBad3661786f3a/jU9qUqkhF5PAG_TXIB0Mb481-cGaaaaTAvAF8FaHt40>

<https://connect2id.com/products/nimbus-jose-jwt/examples/jwt-with-es256k-signature>

ECDSA 只是签名算法。与 RSA 和 AES 不同，这种算法不能用于加密。以太坊采用的是 secp256k1 曲线。

Actually, it is not possible to uniquely recover the public key from an ECDSA signature (𝑟,𝑠). This remains true even if we also assume you know the curve, the hash function used, and you also have the message that was signed.

<https://crypto.stackexchange.com/questions/18105/how-does-recovering-the-public-key-from-an-ecdsa-signature-work>
