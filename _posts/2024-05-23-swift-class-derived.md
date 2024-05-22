---
layout: post
title: Swift的类函数和继承
subtitle:
categories: Swift iOS
tags: [Swift, iOS]
---

## Swift 的类函数和继承

### 单例

```swift
class Base {
    class func printClassName() {
        print("base")
    }

    class func foo() {
        print("foo")
    }
}

class Derived: Base {
    override class func printClassName() {
        print("derived")
    }
}


func getDerived() -> Base.Type {
    return Derived.self
}

Derived.printClassName() //"derived"
Base.printClassName() //"base"

Derived.foo() //"foo"
Base.foo() //"foo"

getDerived().printClassName() //"derived"
```
