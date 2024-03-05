---
layout: post
title: CocoaPods 配置 Xcode 中的自定义脚本
subtitle:
categories: iOS CocoaPods
tags: [iOS, CocoaPods, Xcode]
---

## CocoaPods 配置 Xcode 中的自定义脚本

可以在 podspec 中加入如下脚本

```podspec
Pod::Spec.new do |s|

  ...

  s.script_phases = [
    {
      :name => 'My Custom Build Script',
      :script => '${PODS_TARGET_SRCROOT}/MyScript.sh',
      :execution_position => :before_compile,
      :always_out_of_data => "1"
    }
  ]
end
```

其中":always_out_of_data" 对应了 Xcode 中脚本的"Based on dependency analysis"选项

:always_out_of_data => "1"代表不勾选此项

这个选项需要 cocoapods 1.14.3 以上才支持
