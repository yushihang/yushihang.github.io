---
layout: post
title: MetaMask学习笔记
subtitle:
categories: Web3 MetaMask
tags: [MetaMask]
---

## MetaMask 学习笔记

MetaMask 通过 BIP39 + BIP32 + BIP44 生成钱包

每个 account 的 address(通过 privatekey 生成) 都可以通过下面的路径推导出来

m/44'/60'/0'/0/{account_index}

所以当使用助记词恢复钱包时，会使用 account_index 从 0 到 19 遍历， 生成 20 个 account ，然后删除没有资产的所有 account，如果所有的账户都没有资产，那么就保留第一个账号

代码参见[链接](https://github.com/MetaMask/metamask-mobile/blob/4dd9073ee0cb4778fa6ecbf44db37b66817bca27/app/util/importAdditionalAccounts.js#L1)

文档参见[链接](https://support.metamask.io/hc/en-us/articles/360015489271-How-to-add-missing-accounts-after-restoring-with-Secret-Recovery-Phrase)
