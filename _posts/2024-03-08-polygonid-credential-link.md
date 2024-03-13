---
layout: post
title: DID 学习日记 - PolygonID DID Wallet (iOS AppStore版本的) 的操作流程体验
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3]
---

## DID 学习日记 - PolygonID DID Wallet (iOS AppStore 版本的) 的操作流程体验

### 相关网页/工具

- issuer: <https://issuer-ui.polygonid.me/credentials/issued>
- schema explorer: <https://schema-builder.polygonid.me/>
- sdr/query builder
- jwz validator: <https://jwz.polygonid.me/>

### 操作方式

两种操作方式

- Direct issue credential
- Credential Link

### Direct issue credential

指定 holder 的 DID 来 issue credential
流程如下:

- 客户端创建钱包
- 获取客户端的 DID
- issuer 侧 import schema
- 选择 schema + holder DID
- 填入 schema 对应的 claims 信息
- 生成 credential qrcode
- app 扫码, 下载并添加 credential
- 在 query builder 上填入 schema 的 jsonld，并设置查询条件，生成 query qrcode
- app 扫码，查询本地 vc， 生成本地 vp， 上传到 verifier, 完成 vp 验证
- 网页端会有一个 jwz verifier

  #### 如果只选择 MTP 和 Sig

  那么只要 verifier/query builder 选择的验证方式和 VC 生成时一样即可

  #### 如果同时选择了 MTP 和 Sig

  - 在 state publish 之前， 扫描 vc qrcode 只能下载 sig 的签名，下载后在钱包里看到的内容如下

    ![sig vc]({{ "/assets/images/2024-03-08/bjjonly.png" | absolute url }})

    - 可以看到其中的`Proof types` 只有 `BJJSignature2021`

    - 此时如果去验证一个指定了 Sig 模式的 verifier/sdr qrcode, 是可以成功的

    - 但如果去试图验证一个指定了 mtp 模式的 verifier/sdr qrcode, 会遇到如下错误
      ![sig vc]({{ "/assets/images/2024-03-08/mtpverifyerror.png" | absolute url }})

      原因是 polygonID 的 flutter sdk 会根据 schema(hash) 和 type (sig/mtp) 去寻找本地的 vc，此时本地没有 mtp 的 vc，所以报错。

    - 此时去 publish issuer state，等待 20-30s 等 publish 到链上
    - 再去打开 vc 的二维码扫描，并下载 vc，此时可以看到 vc 里已经同时有了 mtp 和 sig 两种
      ![sig vc]({{ "/assets/images/2024-03-08/sigmtp.png" | absolute url }})

    - 这时无论 query qrcode 指定了 sig 或者 mtp，都能正确的生成 vp 并验证。

### Credential Link

- 这种情况下不需要事先指定 DID, 只需要限定能 issue vc 的个数
- holder 扫码后才完成 issue
- vc 生成完毕后，通过 apns 或者 FCM/GCM 将知 app 下载 vc
- 如果用户错过了推送， issuer web 页面上可以再次展示 qrcode 供 app 扫描下载 vc

#### 如果只选择 MTP 和 Sig

- sig 方式会很快收到推送
- mtp 方式需要等 publish 成功后才能收到推送

#### 如果同时选择了选择 MTP 和 Sig

- 用户会先收到 sig 的 vc 推送，下载后看到如下 vc
  ![sig vc]({{ "/assets/images/2024-03-08/bjjonly.png" | absolute url }})

- publish issuers state 后， 会收到另一条推送，revoke 掉之前的 sig vc

  ![sig vc]({{ "/assets/images/2024-03-08/revoke-notification.png" | absolute url }})

  ![sig vc]({{ "/assets/images/2024-03-08/revoke-0.png" | absolute url }})

  ![sig vc]({{ "/assets/images/2024-03-08/revoke-1.png" | absolute url }})

- publish 成功后，会收到第三条推送，下载 sig+mtp vc
