---
layout: post
title: DID 学习日记 - PolygonID - 数据/操作流程 + 协议细节
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3]
---

## DID 学习日记 - PolygonID - 数据/操作流程 + 协议细节

- 获取 schema 列表, 或者 build 一个自己的
  <https://schema-builder.polygonid.me>

- 复制 Schema Url, 例如 ipfs://QmPt89E3QRjbFDHZRKmNXLoTssreHH4uiz44XWNMdPFN4e

- import schema
  <https://issuer-ui.polygonid.me/schemas/import-schema>

- issue credential
  <https://issuer-ui.polygonid.me/credentials/issue>

  - 如果选择了 MTP 类型，需要去<https://issuer-ui.polygonid.me/issuer-state>点击 publish 一下

    把 issuer 的 state 同步到智能合约里(因为插入了 claims,所以 claims tree 变化导致 state 变化了)

- 在<https://issuer-ui.polygonid.me/credentials/issued>找到刚 issue 的 credential, 点击右边的三个点按钮, 选择 Details

- 在打开的<https://issuer-ui.polygonid.me/credentials/issued/xxxxxx>页面中赋值 qrcode link,例如<https://issuer-ui.polygonid.me/credentials/scan-issued/xxxxxx>

- 识别这个二维码, 会得到如下格式的链接:<iden3comm://?request_uri=https://issuer-admin.polygonid.me/v1/qr-store?id=db20325f-15da-41c1-926c-a4c133f389c5>

- 访问其中的https://issuer-admin.polygonid.me/v1/qr-store?id=db20325f-15da-41c1-926c-a4c133f389c5

  内容格式如下

  ```json
  {
    "id": "bff72926-340e-4632-acdb-088f74c89aae",
    "typ": "application/iden3comm-plain-json",
    "type": "https://iden3-communication.io/credentials/1.0/offer",
    "thid": "bff72926-340e-4632-acdb-088f74c89aae",
    "body": {
      "url": "https://issuer-admin.polygonid.me/v1/agent",
      "credentials": [
        {
          "id": "88a96d42-c636-11ee-93b5-0242ac120009",
          "description": "KYCAgeCredential"
        }
      ]
    },
    "from": "did:polygonid:polygon:mumbai:2qMCebtitXNzau92r4JNV3y162hkzVZPn75UPMiE1G",
    "to": "did:polygonid:polygon:mumbai:2qKuqAopNFKhy7s6W9xHN3sP4sVHrDHLwrnTPVYJna"
  }
  ```

- 这个数据可以用来传递到 PolygonID SDK 的 fetchAndSaveClaims 接口中
  <https://devs.polygonid.com/docs/wallet/wallet-sdk/polygonid-sdk/iden3comm/api/fetch-and-save/#fetch-and-save-credentials>

  其中 Iden3MessageEntity 的定义如下

  ```dart
  abstract class Iden3MessageEntity {
  final String id;
  final String typ;
  final String type;
  final Iden3MessageType messageType;
  final String thid;
  abstract final body;
  final String from;
  final String? to;

  const Iden3MessageEntity(
      {required this.id,
      required this.typ,
      required this.type,
      this.messageType = Iden3MessageType.unknown,
      required this.thid,
      required this.from,
      this.to});

  @override
  String toString() =>
      "[Iden3MessageEntity] {id: $id, typ: $typ, type: $type, messageType: $messageType, thid: $thid, body: $body, from: $from, to: $to}";

  @override
  Map<String, dynamic> toJson() => {
        'id': id,
        'typ': typ,
        'type': type,
        'messageType': messageType.name,
        'thid': thid,
        'body': body.toJson(),
        'from': from,
        'to': to,
      }..removeWhere(
          (dynamic key, dynamic value) => key == null || value == null);

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Iden3MessageEntity &&
          runtimeType == other.runtimeType &&
          id == other.id &&
          typ == other.typ &&
          type == other.type &&
          messageType == other.messageType &&
          thid == other.thid &&
          body == other.body &&
          from == other.from &&
          to == other.to;

  @override
  int get hashCode => runtimeType.hashCode;
  }
  ```

- fetchAndSaveClaims 返回的数据为 List\<ClaimEntity\>

  ```dart

  class ClaimEntity {
  final String id;
  final String issuer;
  final String did;
  final ClaimState state;
  final String? expiration;
  final Map<String, dynamic>? schema;
  final String type;
  final Map<String, dynamic> info;

  ClaimEntity(
  {required this.id,
  required this.issuer,
  required this.did,
  required this.state,
  this.expiration,
  this.schema,
  required this.type,
  required this.info});

  factory ClaimEntity.fromJson(Map<String, dynamic> json) {
  return ClaimEntity(
  id: json['id'],
  issuer: json['issuer'],
  did: json['did'],
  state: ClaimState.values.firstWhere((e) => e.name == json['state']),
  expiration: json['expiration'],
  schema: json['schema'],
  type: json['type'],
  info: json['info'],
  );
  }

  @override
  Map<String, dynamic> toJson() => {
  'id': id,
  'issuer': issuer,
  'did': did,
  'state': state.name,
  'expiration': expiration,
  'schema': schema,
  'type': type,
  'info': info,
  };

  @override
  String toString() => "[ClaimEntity] {id: $id, "
  "issuer: $issuer, did: $did, state: $state, "
  "expiration: $expiration, schema: $schema, type: $type, info: $info}";

  @override
  bool operator ==(Object other) =>
  identical(this, other) ||
  other is ClaimEntity &&
  runtimeType == other.runtimeType &&
  id == other.id &&
  issuer == other.issuer &&
  did == other.did &&
  state == other.state &&
  expiration == other.expiration &&
  schema == other.schema &&
  type == other.type &&
  info.toString() == other.info.toString();

  @override
  int get hashCode => runtimeType.hashCode;
  }
  ```

- 生成 Sdr 的网页为<https://verifier-demo.polygonid.me>

- 生成对应 credential 的 sdr 二维码之后解析二维码

  格式为

  ```json
  {
    "id": "46bd4f92-84ed-4276-a610-f1b4cdcec68e",
    "typ": "application/iden3comm-plain-json",
    "type": "https://iden3-communication.io/authorization/1.0/request",
    "thid": "46bd4f92-84ed-4276-a610-f1b4cdcec68e",
    "body": {
      "callbackUrl": "https://self-hosted-demo-backend-platform.polygonid.me/api/callback?sessionId=77396",
      "reason": "test flow",
      "scope": [
        {
          "id": 1,
          "circuitId": "credentialAtomicQuerySigV2",
          "query": {
            "allowedIssuers": ["*"],
            "context": "https://raw.githubusercontent.com/iden3/claim-schema-vocab/main/schemas/json-ld/kyc-v4.jsonld",
            "credentialSubject": {
              "birthday": {
                "$lt": 20000101
              }
            },
            "type": "KYCAgeCredential"
          }
        }
      ]
    },
    "from": "did:polygonid:polygon:mumbai:2qH7TstpRRJHXNN4o49Fu9H2Qismku8hQeUxDVrjqT"
  }
  ```

- 这个数据可以被送到 Get Proofs 里生成 proof
  <https://devs.polygonid.com/docs/wallet/wallet-sdk/polygonid-sdk/iden3comm/api/get-proofs/#get-proof>

- GetProofs 的返回值为 List\<Iden3commProofEntity\>

  ```dart
  class Iden3commProofEntity extends ZKProofEntity {
    final int id;
    final String circuitId;
  }
  ```

  ```dart
  class ZKProofEntity {
    final ZKProofBaseEntity proof;
    final List<String> pubSignals;
  }
  ```

  ```dart
  class ZKProofEntity {
    final ZKProofBaseEntity proof;
    final List<String> pubSignals;
  }
  ```

  ```dart
  class ZKProofBaseEntity {
    final List<String> piA;
    final List<List<String>> piB;
    final List<String> piC;
    final String protocol;
    final String curve;
  }
  ```

- 这个 proof 还需要加上 authToken 并 encode 为 JWZ

  这个接口 PolygonID 没有直接开发，而是包含在了 authenticate 接口中
  <https://devs.polygonid.com/docs/wallet/wallet-sdk/polygonid-sdk/iden3comm/api/authenticate/>

  也就是说可以把 proof 作为 authenticate 的 message 发送

  也可以自己调用[GetJWZUseCase](https://github.com/0xPolygonID/polygonid-flutter-sdk/blob/develop/lib/iden3comm/domain/use_cases/get_jwz_use_case.dart)来封装
