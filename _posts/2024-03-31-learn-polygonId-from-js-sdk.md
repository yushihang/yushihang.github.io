---
layout: post
title: DID 学习日记 - PolygonID DID JS SDK
subtitle:
categories: DID PolygonID Web3 JavaScript
tags: [DID, PolygonID, Web3, JavaScript]
---

## DID 学习日记 - PolygonID DID JS SDK

### Flutter SDK 的相关库

- <https://github.com/0xPolygonID/polygonid-flutter-sdk/> (Flutter / dart)
- <https://github.com/0xPolygonID/witnesscalc> (C++)
- <https://github.com/iden3/rapidsnark> / <https://github.com/0xPolygonID/rapidsnark> (C++)
- <https://github.com/0xPolygonID/c-polygonid> (Golang)
- <https://github.com/0xPolygonID/polygonid-flutter-sdk/tree/develop/rust> (Rust)

如果不是已经对他们之间的调用关系非常熟悉了，平时去查询各种逻辑处理细节还是有点麻烦，需要在 Flutter 和 C++/Golang/Rust 代码里反复穿梭。

特别是 Golang 的工程还引用了很多 Iden3 的其他 Golang 库, 在确认电路输入参数生成细节时非常麻烦。

### JS-SDK

正好最近在学习 Verifier 的逻辑时，发现<https://github.com/0xPolygonID/js-sdk>里除了有 Verifier 的逻辑，还几乎包含了 Holder 的全套逻辑。

而且很多代码就是在同一个类里实现的， 例如 zkproof 的 generate 和 verify, 可以对照比对逻辑，简直是一个宝藏。

相关学习建议如下:

#### Curcuit Codes

<https://github.com/iden3/circuits>

其中各种 main 函数在<https://github.com/iden3/circuits/tree/master/circuits>目录下

实现细节则在以下目录下

- <https://github.com/iden3/circuits/tree/master/circuits/offchain>
- <https://github.com/iden3/circuits/tree/master/circuits/onchain>
- <https://github.com/iden3/circuits/tree/master/circuits/auth>

电路的文档在 <https://docs.circom.io/>

#### Create Identity (Holder)

```javascript
  /**
   * Create Identity creates Auth BJJ credential,
   * Merkle trees for claims, revocations and root of roots,
   * adds auth BJJ credential to claims tree and generates mtp of inclusion
   * based on the resulting state it provides an identifier in DID form.
   *
   * @param {IdentityCreationOptions} opts - default is did:iden3:polygon:mumbai** with generated key.
   * @returns `Promise<{ did: DID; credential: W3CCredential }>` - returns did and Auth BJJ credential
   * @public
   */

  createIdentity(opts: IdentityCreationOptions): Promise<{ did: DID; credential: W3CCredential }>;

  /**
   * Creates profile based on genesis identifier
   *
   * @param {DID} did - identity to derive profile from
   * @param {number} nonce - unique integer number to generate a profile
   * @param {string} verifier - verifier identity/alias in a string from
   * @returns `Promise<DID>` - profile did
   */
  createProfile(did: DID, nonce: number, verifier: string): Promise<DID>;
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/identity/identity-wallet.ts>

#### Create Credential (Holder)

```javascript
  /**
   * Issues new credential from issuer according to the claim request
   *
   * @param {DID} issuerDID - issuer identity
   * @param {CredentialRequest} req - claim request
   * @returns `Promise<W3CCredential>` - returns created W3CCredential
   */
  issueCredential(issuerDID: DID, req: CredentialRequest, opts?: Options): Promise<W3CCredential>;

```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/identity/identity-wallet.ts>

```javascript
  /**
   * Creates a W3C verifiable Credential object
   *
   * @param {string} hostUrl - URL that will be used as a prefix for credential identifier
   * @param {DID} issuer - issuer identity
   * @param {CredentialRequest} request - specification of claim creation parameters
   * @param {JSONSchema} schema - JSON schema for W3C Verifiable Credential
   * @returns W3CCredential
   */
  createCredential(issuer: DID, request: CredentialRequest, schema: JSONSchema): W3CCredential;
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/credentials/credential-wallet.ts>

#### Handle SDR / ZKProof request (Holder)

```javascript
  /**
   * Processes zero knowledge proof requests.
   *
   * @param senderIdentifier - The identifier of the sender.
   * @param requests - An array of zero knowledge proof requests.
   * @param from - The identifier of the sender.
   * @param proofService - The proof service.
   * @param opts - Additional options for processing the requests.
   * @returns A promise that resolves to an array of zero knowledge proof responses.
   */
  export const processZeroKnowledgeProofRequests = async (
    senderIdentifier: DID,
    requests: ZeroKnowledgeProofRequest[] | undefined,
    from: DID | undefined,
    proofService: IProofService,
    opts: {
      mediaType?: MediaType;
      packerOptions?: JWSPackerParams;
      supportedCircuits: CircuitId[];
      ethSigner?: Signer;
      challenge?: bigint;
    }
  ): Promise<ZeroKnowledgeProofResponse[]>
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/iden3comm/handlers/common.ts>

此函数可以展开为以下逻辑

##### Generate Circuit Inputs (Holder)

```javascript
  /**
   * generates auth inputs
   *
   * @param {Uint8Array} hash - challenge that will be signed
   * @param {DID} did - identity that will generate a proof
   * @param {Number} profileNonce - identity that will generate a proof
   * @param {CircuitId} circuitId - circuit id for authentication
   * @returns `Promise<Uint8Array>`
   */
  generateAuthV2Inputs(hash: Uint8Array, did: DID, circuitId: CircuitId): Promise<Uint8Array>;
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/proof/proof-service.ts>

```javascript
  private credentialAtomicQueryMTPV2PrepareInputs = async ({
    preparedCredential,
    identifier,
    proofReq,
    params,
    circuitQueries
  }: InputContext): Promise<Uint8Array>
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/proof/provers/inputs-generator.ts>

```javascript
  private credentialAtomicQuerySigV2PrepareInputs = async ({
    preparedCredential,
    identifier,
    proofReq,
    params,
    circuitQueries
  }: InputContext): Promise<Uint8Array>
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/proof/provers/inputs-generator.ts>

##### Generate & Verify Zkproof (Holder / Verifier)

```javascript
/**
 * ZKProver is responsible for proof generation and verification
 *
 * @public
 * @interface ZKProver
 */
export interface IZKProver {
  /**
   * generates zero knowledge proof
   *
   * @param {Uint8Array} inputs - inputs that will be used for proof generation
   * @param {string} circuitId - circuit id for proof generation
   * @returns `Promise<ZKProof>`
   */
  generate(inputs: Uint8Array, circuitId: string): Promise<ZKProof>;
  /**
   * verifies zero knowledge proof
   *
   * @param {ZKProof} zkp - zero knowledge proof that will be verified
   * @param {string} circuitId - circuit id for proof verification
   * @returns `Promise<boolean>`
   */
  verify(zkp: ZKProof, circuitId: string): Promise<boolean>;
}
```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/proof/provers/prover.ts>

#### Read Public Signals from Zkproof (Verifier)

```javascript
  /**
   * PubSignalsUnmarshal unmarshal auth.circom public inputs to AuthPubSignals
   *
   * @param {Uint8Array} data
   * @returns AuthV2PubSignals
   */
  pubSignalsUnmarshal(data: Uint8Array): AuthV2PubSignal

  /**
   * PubSignalsUnmarshal unmarshal credentialAtomicQueryMTP.circom public signals array to AtomicQueryMTPPubSignals
   *
   * @param {Uint8Array} data
   * @returns AtomicQueryMTPV2PubSignals
   */
  pubSignalsUnmarshal(data: Uint8Array): AtomicQueryMTPV2PubSignals

  /**
   *
   * PubSignalsUnmarshal unmarshal credentialAtomicQuerySig.circom public signals array to AtomicQuerySugPubSignals
   * @param {Uint8Array} data
   * @returns AtomicQuerySigV2PubSignals
   */
  pubSignalsUnmarshal(data: Uint8Array): AtomicQuerySigV2PubSignals

```

<https://github.com/0xPolygonID/js-sdk/tree/main/src/circuits>

#### Verify Public Signals (Verifier)

```javascript
  private authV2Verify = async ({
    sender,
    challenge,
    pubSignals,
    opts
  }: VerifyContext): Promise<BaseConfig>

  private credentialAtomicQueryMTPV2Verify = async ({
    query,
    verifiablePresentation,
    sender,
    challenge,
    pubSignals,
    opts
  }: VerifyContext): Promise<BaseConfig>

  private credentialAtomicQuerySigV2Verify = async ({
    query,
    verifiablePresentation,
    sender,
    challenge,
    pubSignals,
    opts
  }: VerifyContext): Promise<BaseConfig>

```

<https://github.com/0xPolygonID/js-sdk/blob/main/src/proof/verifiers/pub-signals-verifier.ts>
