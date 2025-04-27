---
layout: post
title: 我所理解的Javascript prototype chain
subtitle:
categories: FrontEnd Web Javascript Javascript
tags: [FrontEnd, Web, Javascript, Javascript]
---

## 我所理解的 Javascript prototype chain

### Objective-C 中的类继承关系图

![class-relation-oc]({{ "/assets/images/2025-04-27/Objective-C_metaclass.png" | absolute url }})

对 iOS 开发熟悉的同学应该都知道，OC 中有个经典的 metaclass / 和类继承关系图

### Javascript 中的 prototype chain

而在 JS 中, 也同样存在类似的 prototype chain, 如下图所示
![js-proto-chain]({{ "/assets/images/2025-04-27/js-prototype-chain.jpg" | absolute url }})

这个图看起来比较复杂，我们试着来拆解一下

### prototype 和 **proto** 的区别

1. **prototype**：

   - 仅存在于函数对象上（构造函数）
   - 用于实现基于原型的继承
   - 示例：`Function.prototype`, `Array.prototype`

2. \_\_proto\_\_：
   - 存在于所有对象实例上
   - 指向构造函数的 prototype 属性
   - 现代代码建议使用`Object.getPrototypeOf()`

### Object 和 Function

在 JavaScript 中，Object 和 Function 有着特殊的相互关系：

1. **Object**:

   - 是所有对象的基类
   - 通过`Object.prototype`提供基础方法
   - 用`{}`创建的对象都继承自`Object.prototype`
   - Object.prototype.\_\_proto\_\_是 null,为继承关系的终点

2. **Function**:

   - 是函数的构造函数
   - 所有函数都是`Function`的实例
   - 包含`call()`,`apply()`等方法
   - 所有构造函数的 \_\_proto\_\_ 都指向 Function.prototype

   <br>

   Function 和 Object 既是函数也是对象

关键关系：

1.  prototype.constructor

    - `Function.prototype.constructor === Function`
    - `Object.prototype.constructor === Object`

2.  Function 的 prototype 和 \_\_proto\_\_

    - `Function.__proto__ === Function.prototype`

3.  Object 和 Function 的关系
    - `Function.prototype.__proto__ === Object.prototype`
    - `Object.__proto__ === Function.prototype`

### prototype chain 图示

- 图 1:

![js-proto-chain]({{ "/assets/images/2025-04-27/obj-fun.jpg" | absolute url }})

- 图 2:
  ![js-proto-chain]({{ "/assets/images/2025-04-27/obj-fun-1.jpg" | absolute url }})
