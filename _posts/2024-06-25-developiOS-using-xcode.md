---
layout: post
title: 使用VSCode开发iOS项目
subtitle:
categories: iOS Xcode VSCode
tags: [iOS, Xcode, VSCode]
---

## 使用 VSCode 开发 iOS 项目

### 1. 摆脱对 xcodeproj 文件编辑的需求。

使用 XcodeGen 或者 CocoaPods 之类的工具实现，所有的 xcodeproj 和 xcworkspace 文件都根据文件目录架构和配置文件重新生成。

### 2. 创建编译 Task

#### task.json

在项目根目录下的.vscode 目录里新建一个 task.json
格式如下

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build",
      "type": "shell",
      "command": "xcodebuild -project xxxx.xodeproj -schema xxx -configuration xxx -sdk xxx -arch xxx build | xcbeautify",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "reveal": "always"
      }
    }
  ]
}
```

其中 xcbeautify 是一个可选的命令行工具，用于美化 Xcode 的构建输出，需要确保它已安装,并且可以在 terminal 运行（可以使用 brew install xcbeautify 安装）。

#### json 中的 build 类型

```json
      "group": {
        "kind": "build",
        "isDefault": true
      },
```

留意这里的 "kind": "build"

我们在 vscode 里按下 shift+cmd+p, 然后输入 keyboard,并选择 Open Default Keyboard Shortcuts (JSON)

在打开的 Default Keybindings 文件中可以看到如下内容

```json
[
  ...

  {
    "key": "shift+cmd+b",
    "command": "workbench.action.tasks.build",
    "when": "taskCommandsRegistered"
  },

  ...
]
```

然后我们就可以使用 shift+cmd+b 来运行 build task,
其实就是执行 "command": "xcodebuild -project xxxx.xodeproj -schema xxx -configuration xxx -sdk xxx -arch xxx build | xcbeautify", 编译结果会在 VSCode 的输出中实时显示

### 3. 使用 iOS-debug extension 进行调试

#### 编译并安装插件

<https://github.com/nisargjhaveri/vscode-ios-debug>

这个插件推荐使用自己编译的版本，因为有个 PR 我提交给作者后，作者 merge 后，还没有发布到最新版本中。

PR 地址为 <https://github.com/nisargjhaveri/vscode-ios-debug/pull/22>

编译过程如下

a. git clone https://github.com/nisargjhaveri/vscode-ios-debug

b. cd vscode-ios-debug

c. npm install -g vsce

d. vsce package

e. install generated ios-debug-0.4.0.vsix

#### 编辑 launch.json

在项目根目录下新建一个 .vscode/launch.json 文件，内容如下(或者编辑原有文件)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug xxx App",
      "program": "${workspaceFolder}/xxx/xxx/xxx.app",
      "iOSBundleId": "com.xxx.xxx",
      "iosTarget": "last-selected",
      "cwd": "${workspaceFolder}",
      "internalConsoleOptions": "openOnSessionStart",
      "console": "internalConsole",
      "sourceMap": {
        "./": "${workspaceFolder}"
      },
      "preLaunchTask": "build"
    }
  ]
}
```

#### 配置项详细解释

- version: "0.2.0"

  这是 launch.json 配置文件的版本号。

- configurations:

  这是一个包含所有调试配置的数组，每个调试配置都用一个对象表示。

- type: "lldb"

  指定调试器的类型，这里使用的是 LLDB 调试器。

- request: "launch"

  指定调试请求的类型，这里是启动调试会话。

- name: "Debug xxx App"

  这是调试配置的名称，用于在 VS Code 中标识这个调试配置。

- program: "\${workspaceFolder}/xxx/xxx/xxx.app"

  指定要调试的程序的路径。\${workspaceFolder} 是工作区文件夹的路
  径，xxx/xxx/xxx.app 是相对于工作区文件夹的可执行文件路径。

  此处要注意把每次编译生成的 App 从 Xcode 的 DerivedData 目录 copy 到配置的目录中
  例如通过如下方式进行

  ```
  SKIP_INSTALL = NO
  DEPLOYMENT_POSTPROCESSING = YES
  DEPLOYMENT_LOCATION = YES
  INSTALL_PATH = $(CONFIGURATION)-$(PLATFORM_NAME)/$ (PRODUCT_NAME:c99extidentifier)
  DSTROOT = $(SCROOT)/xxx/
  ```

- iOSBundleId: "com.xxx.xxx"

  指定要调试的 iOS 应用程序的 Bundle Identifier。

- iosTarget: "last-selected"

  指定要调试的 iOS 目标设备。"last-selected" 表示使用上次选中的设备。也可以和 Flutter 工程一样，直接在 VSCode 的 bottombar 中选择当前设备

- cwd: "\${workspaceFolder}"
  指定调试会话的当前工作目录。\${workspaceFolder} 是工作区文件夹的路径。

- internalConsoleOptions: "openOnSessionStart"

  指定内部控制台的选项。"openOnSessionStart" 表示在调试会话开始时打开内部控制台。

- console: "internalConsole"

  指定要使用的控制台类型。"internalConsole" 表示使用 VS Code 的内部控制台。

- sourceMap:

  这是一个源文件映射对象，用于将调试信息中的源文件路径映射到工作区中的路径。

  "./": "\${workspaceFolder}"

  表示将当前目录映射到工作区文件夹。

- preLaunchTask: "build"

  指定在启动调试会话之前要运行的任务。"build" 是之前在 tasks.json 文件中定义的构建任务。

#### 开始调试

按下 F5 或者使用 VSCode 的调试 UI 按钮开始调试。

调试会话开始后，VS Code 会自动打开一个新的调试控制台，并开始输出调试信息。

断点 / 运行堆栈 / watch 都是可用的

### 4. 代码自动完成提示 & 符号自动跳转

我们需要把代码的 AST(抽象语法树)的数据提交给 VSCode, 有一个开源代码库实现了这个功能

<https://github.com/SolaWing/xcode-build-server>

这个库可以把编译后的 .pcm 文件提交给 VSCode, 然后 VSCode 就可以根据 AST 提供代码自动完成提示和符号自动跳转功能。

#### 修改 task.json

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build",
      "type": "shell",
      "command": "${workspaceFolder}/xxx/xcode-build-server config -project xxxx.xodeproj -schema xxx; xcodebuild -project xxxx.xodeproj -schema xxx -configuration xxx -sdk xxx -arch xxx build | xcbeautify",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "reveal": "always"
      }
    },
    {
      "label": "clean",
      "type": "shell",
      "command": "xcodebuild clean -project xxxx.xodeproj -schema xxx | xcbeautify",
      "group": {
        "kind": "none",
        "isDefault": true
      },
      "presentation": {
        "reveal": "always"
      }
    }
  ]
}
```

可以看到我们增加了一个 clean 的 task (这里一起带上了，跟本小节没有直接关系)

另外我们在原来的 build task 里, 把 command 修改为了

`${workspaceFolder}/xxx/xcode-build-server config -project xxxx.xodeproj -schema xxx; xcodebuild -project xxxx.xodeproj -schema xxx -configuration xxx -sdk xxx -arch xxx build | xcbeautify`

这里的作用就是每次 build 都会触发 xcode-build-server 去工作一次， 当然也可以配置一个单独的 task 来触发 xcode-build-server。

另外此处的 xcode-build-server 看起来是一个可执行文件， 但其实他是一个 python 源码文件，可以直接用 vscode 打开查看。

### 5. 代码 HotReload

参考 <https://github.com/johnno1962/InjectionIII> 文档配置
(待补充)
