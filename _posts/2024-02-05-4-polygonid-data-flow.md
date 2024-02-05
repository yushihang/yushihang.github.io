---
layout: post
title: DID 学习日记 - PolygonID - 从 Secret 到 ProfileDID
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3]
---

## 从 Secret 到 ProfileDID

[https://github.com/0xPolygonID/polygonid-flutter-sdk](https://github.com/0xPolygonID/polygonid-flutter-sdk])

![data flow]({{ "/assets/images/2024-02-05/polygonid-data-flow.jpg" | absolute url }})

ProfileDID 的生成在 Iden3 的 AuthV2 和 MTPV2 / SigV2 都有实现。

我的理解是他可以让 Holder 通过自己定义的不同 nonce，在 GenesisDID 唯一的情况下， 通过 hash 生成不同的 ProfileDID, 这样在面对不同的 issuer 和 verifier 时，可以使用不同的 ProfileDID，从而保护自己的隐私。

PolygonID 的 SDK 在 getproofs 过程中寻找 claims 时，如果 ProfileDID 找不到对应的 claim 也会去找 GenesisDID 拥有的 claim。

此外如果需要接入 BIP39 的 Mnemonic, 也可以从 BIP39 生成的 seedHex 转换为 PolygonID 需要的 secert。

BIP39 的 seedHex 为 512 位，BIP32 的 rootKey 也是 512 位。

但 PolygonID 的实现中，secret(BIP32 的 rootKey) 为 256 位，所以需要将 BIP39 的 seedHex 通过 HMAC-SHA256 转换为 256 位的 secret， 或者是直接取前面的 256 位(Master Private Key)。
