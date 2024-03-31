---
layout: post
title: DID 学习日记 - PolygonID DID VC/VP/Issuer Claim的 Revocation判断
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3, Revocation]
---

## DID 学习日记 - PolygonID DID VC/VP/Issuer Claim 的 Revocation 判断

### Revocation 的定义

- 过期时间
  VC 里存在过期时间的判断，Holder 和 Verifier 都可以通过与标准时间的比对来判断是否过期

- Issuer 主动 revoke
  - Issuer revoke 某一个 Credential
  - Issuer 更换自己的 PrivateKey, 导致 AuthClaim 更新，之前颁发的 Credential 应该都会失效

### Issuer 主动 revoke 的情况

- #### RevocationStatus 数据

  Holder 从 Issuer 处获得的 VC 里保存了 RevocationStatus 的信息，大致如下

  ```json
  {
    "credentialStatus": {
      "id": "https://rhs-staging.polygonid.me/node?state=xxxxxxxx",
      "revocationNonce": 12345678,
      "type": "Iden3ReverseSparseMerkLeTreeProof",
      "statusIssuer": {
        "id": "https://issuer-admin.polygonid.me/v1/credentials/revocation/status/12345678",
        "revocationNonce": 12345678,
        "type": "SparseMerkLeTreeProof"
      }
    }
  }
  ```

  如果 VC 的 proof 数组 里有 Sig, 即 BJJSignature2021 类型的 proof，那么这个 credentialStatus 会同时存在两份。

  一份是 Sig 的 一份是 IssuerClaim 的。

  （因为 Sig 类型的不会进入 Issuer 的 ClaimTree, 所以无法从链上数据得到 Sig 的是否被 Revoke 了）

- #### MTP 类型的 VC 被 Issuer revoke 之后，或者 issuer 的 AuthClaim 发生变换后

  因为 Issuer 的 三颗树的状态肯定会随着发生变化。

  所以只要 Verifier 用电路 public signal 对应 issuer tree state 的 hash 值去链上校验即可。

  具体如下:

  - ##### 电路代码

    无论是 MTPV2 还是 SigV2 的电路，都有一个名为 issuerClaimNonRevState 的 public signal

    <https://github.com/iden3/circuits/blob/696f51cbf126cefa106c3a86a1057918824c393f/circuits/credentialAtomicQuerySigV2.circom#L13C24-L13C46>

    ```circom
    component main{public [requestID,
                        issuerID,
                        issuerClaimNonRevState, //这里
                        claimSchema,
                        slotIndex,
                        claimPathKey,
                        claimPathNotExists,
                        operator,
                        value,
                        timestamp, isRevocationChecked]} = credentialAtomicQuerySigOffChain(40, 32, 64);
    ```

    这个 issuerClaimNonRevState 通过了电路的校验，和 Holder 中 VC 里保存的 issuer 的 tree 相关的信息一致。

    然后 Verifier 进行了如下处理：

    <https://github.com/0xPolygonID/js-sdk/blob/749fc7b77a22bc7c21cef0c9a968de5b805479f6/src/proof/verifiers/pub-signals-verifier.ts#L699>

    ```javascript
    private checkRevocationState = async (
      issuerID: Id,
      issuerClaimNonRevState: Hash,
      opts: VerifyOpts | undefined
      ) => {
      const issuerNonRevStateResolved = await this.checkRevocationStateForId(
        issuerID,
        issuerClaimNonRevState
      );

      const acceptedStateTransitionDelay =
        opts?.acceptedStateTransitionDelay ?? defaultProofVerifyOpts;

      if (!issuerNonRevStateResolved.latest) {
        const timeDiff =
          Date.now() -
          getDateFromUnixTimestamp(Number(issuerNonRevStateResolved.transitionTimestamp)).getTime();
        if (timeDiff > acceptedStateTransitionDelay) {
          throw new Error('issuer state is outdated');
        }
      }
    };
    ```

    这里的 checkRevocationStateForId 的细节可以自己展开，大致逻辑就是去链上获取了 issuer publish 过去的 state hash 值。

    这样可以避免 Holder 恶意修改电路输入，试图欺骗 Verifier。

- #### Sig 类型的 VC 被 Issuer revoke 之后

  如同前面提到的，因为 Sig 的 VC 不会进入 Issuer 的 Auth Claim tree。

  在 Issuer revoke 了 Sig 的 VC 之后（其实 Issuer 的时候也一样）， 对 Holder 和 Verifier 来说，链上没有任何数据发生变化。

  虽然 Holder 可以通过如下 credentialStatus 的 id + revocationNonce 来获取自己这里的 VC 是否过期的信息。

  ```json
  {
    "credentialStatus": {
      "id": "https://rhs-staging.polygonid.me/node?state=xxxxxxxx",
      "revocationNonce": 12345678,
      "type": "Iden3ReverseSparseMerkLeTreeProof",
      "statusIssuer": {
        "id": "https://issuer-admin.polygonid.me/v1/credentials/revocation/status/12345678",
        "revocationNonce": 12345678,
        "type": "SparseMerkLeTreeProof"
      }
    }
  }
  ```

  但是对于 Verifier 来说， 肯定不能相信 Holder 侧做的检测。

  例如有恶意的 Holder 明明自己到自己的 VC 已经被 revoke 了，但是仍然用旧的 RHS 数据生成一个看起来 VC 状态正常的电路 witness， 此时 Verifier 怎么发现问题？

- #### 疑问

  因为 VP 里面并没有 credentialStatus 的数据，无论 mtp 和 sip

  所以无论 issuer 采用 RHS 还是 Onchain 的做法<https://devs.polygonid.com/docs/issuer/features#revocation-status>

  似乎都无法规避恶意的 Holder？

- #### 补充
  RHS 的文档在这里 <https://docs.iden3.io/services/rhs/>
