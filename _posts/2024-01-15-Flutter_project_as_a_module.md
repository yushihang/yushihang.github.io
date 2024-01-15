---
layout: post
title: Flutter作为module集成到iOS/Android项目中的一些问题
subtitle: 一些踩坑记录
categories: Mobile
tags: [Flutter, iOS, Android]
---

## Flutter 作为 module 集成到 iOS/Android 项目中的一些问题

### Flutter 工程的分类

Flutter 工程可以分为两大类，App/Modules 和 Packages

#### Packages

- Dart Packages
- Plugin packages
- FFI Plugin package

官方的定义见[这里](https://docs.flutter.dev/packages-and-plugins/developing-packages#types)

通俗点说，Packages 就是一些可以被其他 Flutter 工程引用的代码库，可以是纯 Dart 代码的 Dart Packages，也可以是包含了 iOS/Android 代码的 Plugin packages。

更特殊的一种的是 FFI Plugin package, 他通过调用 c-style 接口的方式，实现了 Flutter 与原生代码的交互。

类比的话，普通的 Plugin packages 采用的是 Flutter Channel / Method Channel.

#### App / Modules

Flutter App 工程打包后，可以直接在 iOS/Android 上运行。

而 Flutter Modules 可以被 iOS/Android 工程引用，作为一个模块集成到 iOS/Android 工程中 (类似动态库)。

### Flutter Module 的编译方法

类似 App, Flutter Module 需要创建一个 main.dart 文件，需要和 App 一样有自己的入口函数。

参见这个[例子](https://github.com/0xPolygonID/polygonid-flutter-sdk/blob/develop/polygonid-flutter-wrapper/lib/main.dart)

### Flutter Module 的入口函数

默认的入口函数和 app 一样，为 void main()

也可以定义自己的入口函数，函数签名和 main 一样，可以没有参数, 也可以有一个参数，参数类型为 List<String>?

(和其他语言的入口函数一样)

#### 如果自定义了入口函数, 那么需要在函数前面加上 @pragma('vm:entry-point') 注解

否则在执行`flutter build ios-framework`后如果使用了 Release 目录下的 xcframeworks, init() 函数将被剔除掉，导致该 Flutter Module 无法成功运行。

### Flutter Module 的编译方法

- iOS

  ```bash
  flutter build ios-framework
  ```

- Android
  ```bash
  flutter build aar
  ```

### Flutter Module 在 iOS / Android 侧的运行

- `main.dart` 代码如下

  ```dart
  /// Initialize the Flutter SDK wrapper
  /// This method is called from the native side
  @pragma('vm:entry-point')
  Future<void> init(List? env) {
  WidgetsFlutterBinding.ensureInitialized();

  return PolygonIdSdk.init(
      env: env != null && env.isNotEmpty
          ? EnvEntity.fromJson(jsonDecode(env[0]))
          : null);
  }
  ```

- iOS

  ```swift
  let flutterEngine = FlutterEngine(name: "xxxx")
  flutterEngine.run(
            withEntrypoint: "init",
            libraryURI: "package:polygonid_flutter_wrapper/main.dart",
            initialRoute: nil,
            entrypointArgs: [jsonString]
        )
  GeneratedPluginRegistrant.register(with: flutterEngine);
  ```

- Android
  ```kotlin
  val flutterEngine = FlutterEngine(context)
  Handler(Looper.getMainLooper()).post {
    flutterEngine.dartExecutor.executeDartEntrypoint(
      DartExecutor.DartEntrypoint(
        "main.dart",
        "init"
      ),
      env
    )
  }
  ```

注意 Android 的`main.dart`和`init`和 dart 代码的对应

以及 iOS 的`main.dart`需要写为带`package`的全路径。

此外: Release 模式下编译的 Flutter Module xcframework, 因为 Release 模式对 dart VM 做了裁剪， 使用 AOT 模式，不再支持 JIT , 所以无法在 iOS 模拟器上运行，这点需要注意。

参见 [Introduction to Dart VM](https://chromium.googlesource.com/external/github.com/dart-lang/sdk/+/refs/tags/3.2.0-119.0.dev/runtime/docs/)

### Method Channel 的创建

参见 [Method Channel in Flutter: Bridging Native Code and Flutter with Two-Way Communication](https://medium.com/@iiharish97ii/method-channel-in-flutter-bridging-native-code-and-flutter-with-two-way-communication-788d1e91c8c1)

### Flutter Engine 运行时要注意的细节

```objectivec
@interface FlutterEngine : NSObject <FlutterPluginRegistry>

- (BOOL)runWithEntrypoint:(nullable NSString*)entrypoint
               libraryURI:(nullable NSString*)libraryURI
             initialRoute:(nullable NSString*)initialRoute
           entrypointArgs:(nullable NSArray<NSString*>\*)entrypointArgs;
@end
```

这个函数只要文件和入口函数存在就会返回 YES, 但是 Flutter Engine 的启动在业务逻辑上应该是一个异步的操作。

例如 entrypoint 函数运行出现异常。或者前面提到的 Release 模式下的 xcframework 无法在 iOS 模拟器上运行。

很明显这个函数没有提供回调函数，所以无法知道 Flutter Engine 是否启动成功。

此外，在性能比较差的移动设备/模拟器上, 如果在 Flutter Engine 运行后，马上调用 method channel 的函数,也可能导致调用失败。

一种解决方案是在 runWithEntrypoint 函数运行后启动一个定时器，检测 Flutter Engine 的 main isolateID 是否有非空值，如果存在则认为 Flutter Engine 启动成功。

```objectivec
timer = Timer.scheduledTimer(withTimeInterval: 5, repeats: true, block: {
         [weak self] _timer in
          guard let self else {return}

          let hasIsolateId = self.flutterEngine.isolateId != nil


          guard hasIsolateId else {
              print("WARNING! Flutter Engine(PolygonIdSdk) init failed!!!")
              return
          }

          print("Flutter Engine(PolygonIdSdk) init success")

          _timer.invalidate()

          self.methodChannel.invokeMethod("switchLog", arguments: ["enabled": true])
          self.methodChannel.invokeMethod("changeLogLevel", arguments: ["level": "verbose"])


      })
```

### FlutterError 的细节获取

dart 的 MethodChannel 在执行抛出异常时, 会将异常信息包装成 FlutterError。

但是 FlutterError 的 message 字段只包含了异常的类型, 并没有包含异常的详细信息。例如"Instance of xxxxxException"

可以采用如下方案解决

```dart

abstract class CustomException implements Exception {
  StackTrace? _stackTrace;
  CustomException([StackTrace? stackTrace]) {
    _stackTrace = stackTrace ?? StackTrace.current;
  }

  String exceptionInfo() {
    return "";
  }

  String toString() {
    return "[Flutter Exception]: $runtimeType\n[info]:\t${exceptionInfo()} \n[StackTrace]:\n$_stackTrace\n";
  }
}
```

然后在各种派生 Exception 中定义自己的 exceptionInfo 函数

```dart
class XXXXDerivedException extends CustomException {
  final XXXXInfo xxxx;

  XXXXDerivedException(this.xxxx);

  @override
  String exceptionInfo() {
    return "xxxx: $xxxx";
  }
}
```

这样，当 dart 代码在 method channel 的处理函数中执行了`throw XXXXDerivedException(xxxx)`后。

FlutterError 的 message 字段就会包含异常的 exceptionInfo 信息, 以及 stacktrace 详细堆栈。
