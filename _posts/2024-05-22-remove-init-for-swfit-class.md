---
layout: post
title: 让 Swift 的 init()函数无法调用
subtitle:
categories: Swift iOS
tags: [Swift, iOS]
---

## 让 Swift 的 init()函数无法调用

### 单例

```swift
class Singleton {
  static let shared = Singleton()
  private init() {

  }
}
```

我们通常使用的单例如以上代码所示

但是如果我们想声明一个只包含 static 和 class 函数的 swift class, 这时候我们可能希望连自己都无法调用自己的 init 函数

### 关闭 init 函数的实现

![关闭 init] ({{ "/assets/images/2024-05-22/remove_init.jpg" | absolute url }})

```swift
class MyClass {
    @available(*, unavailable, message: "do not call my init()")
    init() {
        fatalError("This initializer is unavailable.")
    }

}

MyClass()

```
