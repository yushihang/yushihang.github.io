---
layout: post
title: DID 学习日记 - PolygonID - JWZ 中 Message 的校验
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3]
---

## JWZ 中 Message 的校验

[https://github.com/0xPolygonID/polygonid-flutter-sdk](https://github.com/0xPolygonID/polygonid-flutter-sdk])

![data flow]({{ "/assets/images/2024-02-05/auth-challange-verification-at-iden3-verifier-side.jpg" | absolute url }})

JWZ 中的 authToken 是通过让 bjjwallet 的 private key 对 message 的 jwz encode + poseidon hash 之后的 challenge 的签名，来完成的对私钥的验证。

这部分验证逻辑是在 AuthV2.circom 中完成的。

而因为 message 可能过大, 所以需要对 message 进行 poseidon hash, 然后再进行签名。

而 message 和 challenge 的一致性校验，是在上图的服务器逻辑中完成的。 这样就保证了 message 不会被篡改。
