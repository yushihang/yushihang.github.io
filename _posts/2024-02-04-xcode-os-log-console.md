---
layout: post
title: Xcode iOS 真机运行日志输出
subtitle:
categories: iOS
tags: [Xcode, iOS, os_log]
---

## Xcode iOS 真机运行日志输出

从某个版本开始, iOS 真机运行时, 从 macOS Consolg app 的窗口不能再看到 Swift 代码使用 print 打印的日志。

经过测试需要用如下代码打印

```swift
import os
os_log("XXXX: \(message, .privacy: .public)")
```

如果只写 os_log("XXXX: \(message)"), macOS Console app 上只能看到"XXXX: \<privacy\>"的打印。
