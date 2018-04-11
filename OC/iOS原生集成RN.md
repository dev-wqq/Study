前提条件：[搭建 React Native 开发环境](https://reactnative.cn/docs/0.51/getting-started.html#content) 和 [安装 CocoaPods ](https://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)

iOS 原生项目集成 React Nactive 具体可以分两个部分：<br>
1.配置环境、依赖库；<br>
2.集成代码。<br>

----
配置环境
----
1.搭建RN开发环境和按照CocoaPods;<br>
2.设置目录结构和 React Native 依赖库；<br>
  * 为了方便管理将所有 RN 相关的文件，放在一个新的文件夹中（例如：RNComponent）；<br>
  * 创建安装 JS 的依赖文件 package.json 和运行安装 React 和 ReactNative；<br>
```
{
  "name": "AppDemo",
  "private": true,
  "dependencies": {
    "react": "^16.3.0",
    "react-native": "^0.54.4"
  },
  "version": "1.0.0",
  "main": "index.js",
  "devDependencies": {},
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "author": "",
  "license": "ISC",
  "description": ""
}
```
```
// 在RNCompoent 目录下执行 npm install 命令安装React和React Native依赖库.
$ npm install
```
3.在Podfile 文件中添加依赖，如果没有使用Cocoapods 管理依赖库，使用 pod init 创建 Podfile 文件.
```
target 'AppDemo' do
  # 将RN相关文档放在RNComponent目录下了，如果不这样操作可以去掉
  # 换成./node_modules/react-native
  pod 'React', :path => './RNComponent/node_modules/react-native', :subspecs => [
    'Core',
    'CxxBridge',  # Include this for RN >= 0.47
    'DevSupport', # Include this to enable In-App Devmenu if RN >= 0.43
    'RCTText',
    'RCTNetwork',
    'RCTWebSocket', # Needed for debugging
    'RCTAnimation', # Needed for FlatList and animations running on native UI thread
    # Add any other subspecs you want to use in your project
  ]
  # 如果使用 RN >= 0.42.0，则需要添加yoga
  pod 'yoga', :path => './RNComponent/node_modules/react-native/ReactCommon/yoga'

  # 第三方 podspec 
  pod 'DoubleConversion', :podspec => './RNComponent/node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
  pod 'glog', :podspec => './RNComponent/node_modules/react-native/third-party-podspecs/glog.podspec'
  pod 'Folly', :podspec => './RNComponent/node_modules/react-native/third-party-podspecs/Folly.podspec'
end
```
```
// 在 Podfile 文件中添加好依赖以后执行 pod install，安装依赖
$ pod install
```
集成代码
----

