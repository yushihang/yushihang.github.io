---
layout: post
title: Flutter开发记录
subtitle: 全局日志打印的截获&从Flutterbundle中读取文件二进制
categories: Mobile Flutter
tags: [Flutter, iOS, Android]
---

## Flutter 开发记录

### 全局日志打印的截获

```dart
Future<void> main() async {
  runZoned(
    () async {
      //在这里写原本的 main 函数中的内容
      WidgetsFlutterBinding.ensureInitialized();

      runApp(const App());
    },
    zoneSpecification: ZoneSpecification(
        print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
      var newLine = LogHelper.getLogString(line); //将 line 的内容按需求进行处理
      parent.print(zone, newLine);
    }),
  );
}
```

### 从 Flutterbundle 中读取文件二进制

可以使用如下语句读取

```dart
  var assets = await rootBundle.load(pathOfAssets);
```

但是这里的 pathOfAssets 不太好确定，特别是 Flutter 工程是作为一个 module 集成到 iOS/Android 工程中的时候。

可以尝试用如下代码获取 bundle 里的所有文件列表，再根据情况处理

```dart
  var assets = await rootBundle.loadString('AssetManifest.json');
  print('AssetManifest.json: $assets');
  Map assetsMap = json.decode(assets);
```
