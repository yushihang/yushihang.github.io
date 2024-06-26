---
layout: post
title: MetaMask学习笔记
subtitle:
categories: Web3 MetaMask
tags: [MetaMask]
---

## MetaMask 学习笔记

### 钱包生成

MetaMask 通过 BIP39 + BIP32 + BIP44 生成钱包

每个 account 的 address(通过 privatekey 生成) 都可以通过下面的路径推导出来

m/44'/60'/0'/0/{account_index}

所以当使用助记词恢复钱包时，会使用 account_index 从 0 到 19 遍历， 生成 20 个 account ，然后删除没有资产的所有 account，如果所有的账户都没有资产，那么就保留第一个账号

代码参见[链接](https://github.com/MetaMask/metamask-mobile/blob/4dd9073ee0cb4778fa6ecbf44db37b66817bca27/app/util/importAdditionalAccounts.js#L1)

文档参见[链接](https://support.metamask.io/hc/en-us/articles/360015489271-How-to-add-missing-accounts-after-restoring-with-Secret-Recovery-Phrase)

### keystore 文件加密解密

逻辑见[这里](https://github.com/trufflesuite/ganache/blob/547c900a50d19b094ef636a9aeccf4f7f2356430/packages/ethereum/ethereum/src/wallet.ts#L342C1-L353C4)

文件例子为

```json
{
  "address": "7e3659eade7759e644984050cc36a47d936f65b9",
  "crypto": {
    "cipher": "aes-128-ctr",
    "ciphertext": "12a84e45881a3744661ded0d6a3414e32d2f0821a76f6699fbc678bed4e69765",
    "cipherparams": {
      "iv": "6cc0c2d5896f2fefad3325c401a64c78"
    },
    "kdf": "scrypt",
    "kdfparams": {
      "dklen": 32,
      "n": 262144,
      "p": 1,
      "r": 8,
      "salt": "aa95cd870512aa5a32a3beca28182bd126535631a40aaa38bd3df302aa07a6e8"
    },
    "mac": "485be04e644faed69c8361349b8891fcd40fed11b08c9dea76dec30f5d1fe2c6"
  },
  "id": "bcfdbf8f-5099-4c09-9eaf-ea8760751619",
  "version": 3
}
```

### 从 BIP39 Seed 生成 BIP32 root key 的逻辑

代码见[这里](https://github.com/yushihang/bip39/blob/de71c22328b24e0848bbe1bd12ac8974ca83b5b8/src/js/index.js#L704C1-L710C6)

```javascript
function calcBip32RootKeyFromSeed(phrase, passphrase) {
  seed = mnemonic.toSeed(phrase, passphrase);
  bip32RootKey = libs.bitcoin.HDNode.fromSeedHex(seed, network);
  if (isGRS())
    bip32RootKey = libs.groestlcoinjs.HDNode.fromSeedHex(seed, network);
}
```

HDNode.fromSeedHex 在最新的[bitcoinjs](http://https://github.com/yushihang/bitcoinjs-lib)

里已经叫做 bip.fromSeed 了

### PolygonID 和 Flutter 的问题

其实 dart 的 bip32 的库是支持 128~512bit 的 seed 的
只是 polygonid 限制了

```dart

  factory BIP32.fromSeed(Uint8List seed, [NetworkType? nw]) {
    if (seed.length < 16) {
      throw new ArgumentError("Seed should be at least 128 bits");
    }
    if (seed.length > 64) {
      throw new ArgumentError("Seed should be at most 512 bits");
    }
    NetworkType network = nw ?? _BITCOIN;
    final I = hmacSHA512(utf8.encode("Bitcoin seed") as Uint8List, seed);
    final IL = I.sublist(0, 32);
    final IR = I.sublist(32);
    return BIP32.fromPrivateKey(IL, IR, network);
  }
```

### ganache

以太坊区块链的本地开发工具

[https://trufflesuite.com/ganache/](https://trufflesuite.com/ganache/)
