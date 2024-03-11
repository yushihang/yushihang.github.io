---
layout: post
title: Xcode 项目中的 strip 设置
subtitle:
categories: iOS Xcode
tags: [Xcode iOS]
---

## 遇到的问题

今天在处理一个 Xcode 项目 archive 后的 ipa 测试时遇到一个问题

我们的项目是 swift 调用 flutter/dart xcframework， 然后 flutter 代码里用 ffi 调用了一些 static lib 里的 c-interface 函数。

因为 flutter 代码打包为 xcframework 时无法直接链接 static library 进去（否则他也不需要用 ffi 方式调用了）

所以这些 static library 最后是打包到了 app 的 binary executable 里面，也就是说 flutter xcframework 里的 dart 逻辑运行时, 会从 app 的代码里试图寻找 ffi 需要的 c-interface 函数。

我们遇到的问题就是说这些函数找不到。

```
[ERROR:flutter/runtime/dart_vm_initializer.cc(41)] Unhandled Exception: Invalid argument(s): Failed to lookup symbol 'someFFIFunc': dlsym(RTLD_DEFAULT, someFFIFunc): symbol not found
```

## 解决方案

因为这些 c-interface 函数在我们 app 里实际没有被使用， clang 的链接器不可能知道 flutter 代码会用 ffi 方式访问他们，所以大概率是被 strip 了。

因此把 build settings 里的 strip style([STRIP_STYLE](https://developer.apple.com/documentation/xcode/build-settings-reference#Strip-Style))从 All Symbols 修改为 Non-Global Symbols 即可

修改后重新 archive, 问题消失

## 为什么只有 Archive 才有问题？

另一个疑问是: 如果我把 Run 和 Archive 的配置都改为 Release, 那么 Run 没问题，Archive 有问题。

这听起来就很不合理，为什么同样的配置，只有 Archive 有问题？

查看文档后发现有以下关联：

strip 流程会受如下配置影响：
Deployment Postprocessing[DEPLOYMENT_POSTPROCESSING](https://developer.apple.com/documentation/xcode/build-settings-reference#Deployment-Postprocessing)

在另一个文档 <https://developer.apple.com/library/archive/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html> 里,提到了默认值如下
YES: When $ACTION = install.
NO: Is the alternative.

而 Archive 会有 ACTION 为 install 阶段（先 build 再 install）， 所以 Archive 会 strip， 而普通的 build 默认不会

新建的 xcode 项目 DEPLOYMENT_POSTPROCESSING 的值为 NO， 但是在 Archive 时也会因为 install 阶段的默认值为 YES 而执行 strip ...

所以如果我们把 DEPLOYMENT_POSTPROCESSING 的值改为 YES, 那么在开发调试阶段也就能提前暴露这类问题， 不用等到 archive 了...

## 补充

<https://developer.apple.com/documentation/xcode/build-settings-reference#Enable-Testability>
Enable Testability(ENABLE_TESTABILITY) 如果设置为 YES ， 也会导致 build 阶段的 strip 不执行, 这个在文档上并未列明。

## 思考

对于 Xcode 默认设置里让开发阶段和 Archive 存在这么大的差异，是不是有点不厚道？
