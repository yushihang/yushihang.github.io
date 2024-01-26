---
layout: post
title: 如何指定生物认证方式(FaceID/TouchID)保护 keychain 数据
subtitle: 指定不能 fallback 到设备密码
categories: iOS
tags: [iOS FaceID]
---

## 如何指定生物认证方式(FaceID/TouchID)保护 keychain 数据

### 如果是手动拉起验证

那么调用[LAContext](https://developer.apple.com/documentation/localauthentication/lacontext)的[evaluatePolicy](https://developer.apple.com/documentation/localauthentication/lacontext/1514176-evaluatepolicy)即可

```swift
func evaluatePolicy(
    _ policy: LAPolicy,
    localizedReason: String,
    reply: @escaping (Bool, (any Error)?) -> Void
)
```

其中[LAPolicy](https://developer.apple.com/documentation/localauthentication/lapolicy)的定义如下:

```swift
case deviceOwnerAuthenticationWithBiometrics
//User authentication with biometry.

case deviceOwnerAuthenticationWithWatch
//User authentication with Apple Watch.

case deviceOwnerAuthenticationWithBiometricsOrWatch
//User authentication with either biometry or Apple Watch.

case deviceOwnerAuthentication
//User authentication with biometry, Apple Watch, or the device passcode.

case deviceOwnerAuthenticationWithWristDetection
//User authentication with wrist detection on watchOS.
```

### 如果使用 keychain 和生物识别的绑定

从 Apple[官方文档](https://developer.apple.com/documentation/localauthentication/accessing_keychain_items_with_face_id_or_touch_id)下载[这个项目](https://docs-assets.developer.apple.com/published/8ae8b636fb/AccessingKeychainItemsWithFaceIDOrTouchID.zip)

修改其中 addCredentials 函数的代码为

```swift
   func addCredentials(_ credentials: Credentials, server: String) throws {

        ...

        let secAccessControlCreateFlags : SecAccessControlCreateFlags = [.biometryCurrentSet, .and ,.devicePasscode]

        // Create an access control instance that dictates how the item can be read later.
        let access = SecAccessControlCreateWithFlags(nil, // Use the default allocator.
                                                     kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly,
                                                     secAccessControlCreateFlags,
                                                     nil) // Ignore any error.


        ...
    }
```

注意其中的 secAccessControlCreateFlags 虽然可以进行多个组合，但是明显矛盾的选项不行。
例如.and 和.or

SecAccessControlCreateFlags 的文档参见[此链接](https://developer.apple.com/documentation/security/secaccesscontrolcreateflags/)
