---
layout: post
title: 在移动操作系统(iOS / Android)中通过设备触碰 NFC 标签打开 App 并进行跳转
subtitle:
categories: Android iOS NFC
tags: [Android iOS NFC]
---

## 在移动操作系统(iOS / Android)中通过设备触碰 NFC 标签打开 App 并进行跳转

### NFC 读取场景

#### 移动端 App 打开后读取 NFC 信息

这个不是我们这篇文章重点讨论的内容，可以参见 Android 和 iOS 的如下文档

Android: [Near field communication] (https://developer.android.com/develop/connectivity/nfc)

iOS: [Building an NFC Tag-Reader App](https://developer.apple.com/documentation/corenfc/building-an-nfc-tag-reader-app)

#### 移动端 App 未打开甚至未安装时，设备读取 NFC 信息后拉起 App 并将数据传递给 App，或是引导用户下载 App

这个是我们这篇文章重点讨论的内容，也是我们在支付宝"碰一下"点餐或者支付时的场景。

##### Android 和 iOS 对 NFC 功能的支持

- 具有 NFC 功能的 Android 设备同时支持两种主要操作模式：

  1. 读写模式

     允许 NFC 设备读写无源 NFC 标签和贴纸。

  2. 模拟卡模式
     允许 Android 设备本身充当 NFC 卡。然后可以由外部 NFC 读取器访问模拟 NFC 卡。

- iOS 对于开发者和第三方 App, 只开放了有限的 NFC 读取模式。

  在 iOS 上，还有一种 NFC 使用场景是通过 Apple Pay+iCloud 支付，但我们这篇文章同样不涉及这个场景

##### 支付宝"碰一下"的使用场景

- Android

  - 必须安装支付宝 App
  - 保持在登录状态
  - 支付宝 App 可以在前台或后台运行，或者为未运行状态。
  - TODO: 暂未测试未安装支付宝 App 时进行碰一下操作的情况
  - 系统需要在解锁状态(待确认)
    用户用 Android 设备触碰 NFC 标签后, 支付宝 App 进入前台, 并通过标签里的 NFC 信息跳转到指定页面完成点餐或者支付操作。

- iOS

  - 如果未安装支付宝 App, 则会引导用户下载支付宝 App
  - 如果安装了 App，需要保持在登录状态
  - 支付宝 App 可以在前台或后台运行，或者为未运行状态。
  - 系统需要在解锁状态(待确认)
    用户用 iOS 设备触碰 NFC 标签后, 会出现一个半屏的支付宝提示，如图:

    ![alipay app clip]({{ "/assets/images/2025-02-16/alipay_appclip.jpg" | absolute url }})

    需要说明的是，这个半屏页面不但会出现在桌面，也会出现在其他前台的 App 上方，甚至是支付宝自己的 App 上方, 如图:

    ![app clip over alipay]({{ "/assets/images/2025-02-16/alipay_appclip_2.jpg" | absolute url }})

    用户可以从这个页面下方跳转 AppStore 安装支付宝 App，也可以点击打开 App 按钮。

    用户选择点击"打开"按钮后

    - 如果用户 iOS 设备上没有安装支付宝 App，会进入如下页面, 按引导下载 App。
      (熟悉 App Clip 的同学应该意识到, 这其实就是一个 App Clip)
      ![download alipay app in app clip]({{ "/assets/images/2025-02-16/alipay_appclip_download_app.jpg" | absolute url }})

    - 如果用户 iOS 设备安装了支付宝 App，则会自动跳转到支付宝 App，进行后续操作
      这应该是绝大多数用户的体验场景，App Clip 自动拉起 App 应该也是 iOS 系统的默认实现。

##### NFC 标签的静态和动态实现

- 静态标签
  NFC 标签内容固定，适用于点餐等场景，可以为贴纸等形态

  ![静态贴纸]({{ "/assets/images/2025-02-16/alipay_nfc_sticker.jpg" | absolute url }})

- 动态标签
  NFC 标签内容动态生成，适用于支付等场景，为带电源的设备

  ![动态设备]({{ "/assets/images/2025-02-16/alipay_nfc_device.jpg" | absolute url }})

##### iOS 侧的技术实现

###### Universal Links / App Links?

还是以支付宝为例，在安装了支付宝的前提下,我们在 iOS 的 Safari 中访问 https://ulink.alipay.com 时,可以看到如下页面:

![Universal Link]({{ "/assets/images/2025-02-16/alipay_safari.jpg" | absolute url }})

可以看到，右上角有一个"打开"按钮，点击后会自动跳转到支付宝 App。并且 iOS 也会自动弹出是否打开支付宝的提示。

我们把这个链接 https://ulink.alipay.com 做成二维码, 然后用 iOS 自带的相机扫描，也会看到如下页面:

![Universal Link]({{ "/assets/images/2025-02-16/alipay_scan.jpg" | absolute url }})

我们点击下方的黄色的"Alipay"提示，就可以自动跳转到支付宝 App。

NFC 标签同样也支持 Universal Links / App Links.

参见如下文档：

- [iOS](https://developer.apple.com/documentation/corenfc/adding-support-for-background-tag-reading)

- [Android](https://developer.android.com/develop/connectivity/nfc/nfc?hl=zh-cn#dispatching)

而如果我们把这个链接写入到 NFC 标签里，用 iOS 设备触碰后，会出现如下提示:

![Universal Link]({{ "/assets/images/2025-02-16/alipay_nfc_url.jpg" | absolute url }})

用户点击 iOS 设备顶部的提示后，也可以跳转到支付宝 App, 这个 url 如果附带了其他 param, 整个 url 的 内容都会被传递给 App。

但这个体验其实跟支付宝现在的体验是不同的，用户没有这个点击 nfc 提示的额外操作。

此外，因为 Google Play Console 在大陆地区无法被普通用户访问，所以 Android App Links 在国内也难以被使用。

###### iOS App Clip

什么是 iOS App Clip，参见<https://developer.apple.com/app-clips/>

对比这个图上的 App Clips 图片和支付宝的半屏图片, 明显看出他们是一样的.
![App Clip]({{ "/assets/images/2025-02-16/app_clips.jpg" | absolute url }})

AppClip 无需实现被安装到用户的手机中，有点类似微信小程序。

AppClip 默认可以通过这类链接被拉起: https://appclip.apple.com/id?p=com.alipay.iphoneclient.clip

当我们把如上链接写入 nfc 芯片后，用 iOS 手机触碰 nfc，就可以实现支付宝目前的效果了(当然支付宝 App 需要传递的具体业务数据我们也需要模拟出来)

此外，近年 Apple 还为 App Clip 提供了一个叫 Advanced App Clip Experiences 的功能，这个功能可以让 App Clip 支持更多的场景, 和支付宝碰一下场景相关的有，用户可以自定义半屏页面的背景图。也可以配置自定义 URL 来写入 NFC 或者通过二维码拉起 App Clip.

<https://developer.apple.com/documentation/appclip/creating-app-clip-codes-with-app-store-connect>

<https://developer.apple.com/help/app-store-connect/offer-app-clip-experiences/offer-an-advanced-app-clip-experience/>

![Advanced App Clip]({{"/assets/images/2025-02-16/advanced_app_clips.jpg" | absolute url }})

![Advanced App Clip]({{"/assets/images/2025-02-16/app_clip_bg.jpg" | absolute url }})

##### 总结如下，我们对 NFC 碰一下的实现技术猜测:

1. iOS NFC 碰一下打开 App
   App Clip + 自定义 URL (基本确认，待具体实现, 需要网站域名支持)

2. Android NFC 碰一下打开 App
   使用 NFC 写入 application/android_appid 实现 (已验证)

3. Android 和 iOS 使用同一个 NFC 芯片
   写入两条数据:

   - Android 的 application/android_app 信息
   - iOS Clip 的 Custom URL + param
     (通过读取支付宝 NFC 标签贴纸确认，动态的支付 NFC 标签未确认)

4. Android 如何读取具体业务需要的 param
   Android 的 application/android_app 信息没有附带额外信息，但是 App 可以主动去读取 NDEF 信息里的第二条数据，也就是通过 iOS Clip 的 Custom URL + param 来获取业务数据 (待实现确认)

##### 补充说明

需要声明一下，我目前没有找到支付宝团队的官方细节剖析，以上的内容都是基于现有文档和可能的技术方案，结合我自己的使用体验进行的推测。

总结： nfc 碰一下是扫码功能的一个替代或者衍生，但是支付宝的碰一下的用户体验做的非常好，其中使用到的技术细节和流程设计非常值得我们学习。

我们是摸着支付宝过河，才把这后面的技术可行路径搞明白，可想而知作为第一个实现碰一下功能的团队，他们在其中进行了多少摸索和尝试取舍。
