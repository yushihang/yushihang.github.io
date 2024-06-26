---
layout: post
title: Web3学习日记 - Wallet Connect 分析 (四)
subtitle:
categories: WalletConnect DApp BlockChain-Wallet
tags: [WalletConnect, BlockChain-Wallet, DApp, ECDHE]
---

## Web3 学习日记 - Wallet Connect 分析 (四)

### DApp 和 Wallet 的通讯数据处理

#### Envelope

WalletConnect 将数据封装方式定义为 Envelope
<https://github.com/WalletConnect/WalletConnectSwiftV2/blob/develop/Sources/WalletConnectKMS/Serialiser/Envelope.swift>

一共有三种 type

```swift
public extension Envelope {
    enum EnvelopeType: Equatable {
        /// type 0 = tp + iv + ct + tag
        case type0
        /// type 1 = tp + pk + iv + ct + tag - base64encoded
        case type1(pubKey: Data) //注意这里 type1需要携带发送方的pubKey
        /// type 2 = tp + base64urlEncoded unencrypted string
        case type2
    }
    ...
}
```

#### 编码处理

对各种 Envelope 的编码处理如下

<https://github.com/WalletConnect/WalletConnectSwiftV2/blob/8c7e413c6f451304235ed161085ddc7947e0e6ae/Sources/WalletConnectKMS/Serialiser/Serializer.swift#L49-L72>

```swift
    /// Encrypts and serializes an object
    /// - Parameters:
    ///   - topic: Topic that is associated with a symetric key for encrypting particular codable object
    ///   - encodable: Object to encrypt and serialize
    ///   - envelopeType: type of envelope
    /// - Returns: Serialized String
    public func serialize(topic: String?, encodable: Encodable, envelopeType: Envelope.EnvelopeType, codingType: Envelope.CodingType) throws -> String {
        if envelopeType == .type2 {
            return try serializeEnvelopeType2(encodable: encodable, codingType: codingType)
        }
        guard let topic = topic else {
            let error = Errors.topicNotFound
            logger.error("\(error)")
            throw error
        }
        let messageJson = try encodable.json()
        guard let symmetricKey = kms.getSymmetricKeyRepresentable(for: topic) else {
            let error = Errors.symmetricKeyForTopicNotFound(topic)
            logger.error("\(error)")
            throw error
        }
        let sealbox = try codec.encode(plaintext: messageJson, symmetricKey: symmetricKey)
        return Envelope(type: envelopeType, sealbox: sealbox, codingType: codingType).serialised(codingType: codingType)
    }
```

可以看到, 编码的时候除了 type2 是明文的, type0 和 type1 都是从 topic 中取出对称加密密钥来对 message 进行加密。
当然 type0 和 type1 在解码时的操作也会有点不同。下面会讲到。

#### 解码

对各种 Envelope 的解码处理如下

<https://github.com/WalletConnect/WalletConnectSwiftV2/blob/8c7e413c6f451304235ed161085ddc7947e0e6ae/Sources/WalletConnectKMS/Serialiser/Serializer.swift#L74-L91>

```swift
    /// Deserializes and decrypts an object
    /// - Parameters:
    ///   - topic: Topic that is associated with a symetric key for decrypting particular codable object
    ///   - encodedEnvelope: Envelope to deserialize and decrypt
    /// - Returns: Deserialized object
    public func deserialize<T: Codable>(topic: String, codingType: Envelope.CodingType, envelopeString: String) throws -> (T, derivedTopic: String?, decryptedPayload: Data) {
        let envelope = try Envelope(codingType, envelopeString: envelopeString)
        switch envelope.type {
        case .type0:
            let deserialisedType: (object: T, data: Data) = try handleType0Envelope(topic, envelope)
            return (deserialisedType.object, nil, deserialisedType.data)
        case .type1(let peerPubKey):
            return try handleType1Envelope(topic, peerPubKey: peerPubKey, sealbox: envelope.sealbox)
        case .type2:
            let decodedType: T = try handleType2Envelope(envelope: envelope)
            return (decodedType, nil, Data())
        }
    }
```

下面我们详细看看每种 Envelope type 的解码细节

##### type2

先看最简单的 type2 处理， 明文传递， 直接 decode 为对应对象
<https://github.com/WalletConnect/WalletConnectSwiftV2/blob/8c7e413c6f451304235ed161085ddc7947e0e6ae/Sources/WalletConnectKMS/Serialiser/Serializer.swift#L133C1-L141C6>

```swift
    private func handleType2Envelope<T: Codable>(envelope: Envelope) throws -> T {
        do {
            let deserialised = try JSONDecoder().decode(T.self, from: envelope.sealbox)
            return deserialised
        } catch {
            logger.error(error)
            throw error
        }
    }
```

##### type0
