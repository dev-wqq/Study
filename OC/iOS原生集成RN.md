前提条件：[搭建 React Native 开发环境](https://reactnative.cn/docs/0.51/getting-started.html#content) 和 [安装 CocoaPods ](https://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)

1.配置环境<br>
 * 搭建RN开发环境和按照CocoaPods；<br>
 * 设置目录结构和 React Native 依赖库；<br>
 * 在Podfile 文件中添加依赖，如果没有使用Cocoapods 管理依赖库，使用 pod init 创建 Podfile 文件。<br>
 
2.集成代码<br>
 * 创建 index.ios.js 文件；<br>
 * 在iOS原生项目导入RN；<br>
 * 运行项目。<br>

3.集成过程遇到的问题及其解决方法<br>
 * 在使用CocoaPods 导入三方依赖的时候由于网络不好(也可能是VPN不稳定)，导入三方依赖库时，经常出现这样的问题；<br>
 * 使用0.55.1版本出现'glog/logging.h' file not found；<br>
 * 'config.h' file not found 和 'glog/logging.h' file not found 如果不是0.55.1版本可以尝试以下命令修复.
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
1.创建 index.ios.js 文件
```
import React, { Component } from 'react';
import {
  AppRegistry,     // 注册
  StyleSheet,      // 样式
  Text,            // 文本组件
  View,            // 视图组件
  Image            // 图片组件
} from 'react-native';

export default class DefaultView extends Component {
    render() {
        return (
            <View style={styles.container}>
              <Text style={styles.welcome}>Hello World!</Text>
              <Text style={styles.instructions}>Welcome to use React Native.</Text>
            </View>
        );
    }
}
// 定制样式
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});
// 注意这里的名字需要和项目名字一致
AppRegistry.registerComponent('AppDemo', () => DefaultView);
```
2.在iOS原生项目导入RN
```
#import <React/RCTRootView.h>
@interface ViewController ()
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    NSString *url = @"http://localhost:8081/index.ios.bundle?platform=ios&dev=true";
    NSURL *jsCode = [NSURL URLWithString:url];
    // 这里的moduleName一定要和下面的index.ios.js里面的注册一样
    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                        moduleName:@"AppDemo"
                                                  nitialProperties:nil
                                                     launchOptions:nil];
    [self.view addSubview:rootView];
    rootView.frame = self.view.bounds;
}
@end
```
3、运行项目
 * 开启 React
```
// 先要启动 React 服务，可以package.json 执行 react-native start，启动React.
// "start": "node node_modules/react-native/local-cli/cli.js start" 设置快捷命令.
$ react-native start
```
 * Xcode 运行项目

----
集成过程遇到的问题及其解决方法
----
1. 在使用CocoaPods 导入三方依赖的时候由于网络不好(也可能是VPN不稳定)，导入三方依赖库时，经常出现这样的问题，参考[error: RPC failed; curl transfer closed with outstanding read data remaining
](https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining/)<br> 
```
// Error Tip:
$ Cloning into 'boost-for-react-native'...
$ remote: Counting objects: 20248, done.
$ remote: Compressing objects: 100% (10204/10204), done.
$ error: RPC failed; curl 18 transfer closed with outstanding read data remaining 
$ fatal: The remote end hung up unexpectedly
$ fatal: early EOF
$ fatal: index-pack failed

// 解决方法：先做一个浅克隆，然后用它去更新 pod 历史仓库
$ git clone http://github.com/boost-for-react-native --depth 1
$ cd boost-for-react-native
$ git fetch --unshallow

PS：--depth用于指定克隆深度，为1即表示只克隆最近一次commit.
as git clone --depth=1 http://github.com/boost-for-react-native 
[error: RPC failed; curl transfer closed with outstanding read data remaining
](https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining/) 
```
2.使用0.55.1版本出现'glog/logging.h' file not found，参考[Fabric should be excluded from Podspec build](https://github.com/facebook/react-native/issues/18683)<br>
```
PS：这里遇到问题是因为导入 React Native 版本是0.55.1存在线上问题，使用0.55.4可以解决这个问题。
Thanks for posting this! It looks like your issue may refer to an older version of React Native. 
Can you reproduce the issue on the latest release, v0.54?

0.55.0 has some issues and we are asking people to stay on 0.54.x for now, which is why the latest release (github.com/facebook/react-native/releases) is no longer 0.55.
```
3、'config.h' file not found 和 'glog/logging.h' file not found 如果不是0.55.1版本可以尝试以下命令修复.
```
$ cd node_modules/react-native/third-party/glog-0.3.4
$ ../../scripts/ios-configure-glog.sh 
```
参考
----
[error: RPC failed; curl transfer closed with outstanding read data remaining
](https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining/)<br> 
[Fabric should be excluded from Podspec build](https://github.com/facebook/react-native/issues/18683)<br>
[解决 git clone 或 git push 太慢的问题](https://blog.minhow.com/2017/03/19/tool/git-slow-solve/)<br>
[React Native 集成现有项目](http://facebook.github.io/react-native/docs/integration-with-existing-apps.html#2-install-javascript-dependencies)

