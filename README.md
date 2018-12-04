# YRuntime
ReactNative 的依赖环境


# 快速集成React Native到现有原生应用(Android版)
## 一.官方集成流程

把 React Native 组件集成到 Android 应用中有如下几个主要步骤：

- 配置好 React Native 依赖和项目结构。
- 创建 js 文件，编写 React Native 组件的 js 代码。
- 在应用中添加一个RCTRootView。这个RCTRootView正是用来承载你的 React Native 组件的容器。
- 启动 React Native 的 Packager 服务，运行应用。
- 验证这部分组件是否正常工作。

这个流程不够通用,集成流程过于复杂,,,,,,所以,写了rn的runtime,集成即可食用
##  二.开发环境准备
### 1. 开发环境准备
#### 安装依赖

必须安装的依赖有：Node、Watchman 和 React Native 命令行工具以及 JDK 和 Android Studio。
#####  1.Node, Watchman

我们推荐使用Homebrew来安装 Node 和 Watchman。在命令行中执行下列命令安装

```
brew install node
brew install watchman

```

Node，需要在 v8.3 以上

Watchman则是由 Facebook 提供的监视文件系统变更的工具。安装此工具可以提高开发时的性能（packager 可以快速捕捉文件的变化从而实现实时刷新）。


##### 2.Yarn、React Native 的命令行工具（react-native-cli）

Yarn是 Facebook 提供的替代 npm 的工具，可以加速 node 模块的下载。React Native 的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务

```
npm install -g yarn react-native-cli

```

##### 3.JDK、Android Studio的安装

略

更多更详细搭建流程 [https://reactnative.cn/docs/getting-started.html](https://reactnative.cn/docs/getting-started.html)

### 2. 配置项目目录结构

首先创建一个空目录用于存放 React Native 项目，然后在其中创建一个/android子目录，把你现有的 Android 项目拷贝到/android子目录中。
### 3.安装 JavaScript 依赖包

- 1) 在项目根目录下创建一个名为package.json的空文本文件，然后填入以下内容：

```
{
  "name": "YReactNativeApp",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  }
}
```
注: 示例中的version字段没有太大意义（除非你要把你的项目发布到 npm 仓库）。scripts中是用于启动 packager 服务的命令。

- 2) 接下来我们使用 yarn 或 npm（两者都是 node 的包管理器）来安装 React 和 React Native 模块。请打开一个终端/命令提示行，进入到项目目录中（即包含有 package.json 文件的目录），然后运行下列命令来安装：
 
 ```
$ yarn add react-native
 ```
 这样默认会安装最新版本的 React Native，同时会打印出类似下面的警告信息（你可能需要滚动屏幕才能注意到）：
 ```
 warning "react-native@0.52.2" has unmet peer dependency "react@16.2.0".
 ```
 
 这是正常现象，意味着我们还需要安装指定版本的 React：
 
 ```
  yarn add react@16.2.0
 ```
 注意必须严格匹配警告信息中所列出的版本，高了或者低了都不可以。
 
 如果你使用多个第三方依赖，可能这些第三方各自要求的 react 版本有所冲突，此时应优先满足react-native所需要的react版本。其他第三方能用则用，不能用则只能考虑选择其他库。
 所有 JavaScript 依赖模块都会被安装到项目根目录下的node_modules/目录中（这个目录我们原则上不复制、不移动、不修改、不上传，随用随装）。

  把node_modules/目录记录到.gitignore文件中（即不上传到版本控制系统，只保留在本地）。
  
### 4. YRuntime 快速集成
#### 1.在项目的 build.gradle 文件中为 React Native 添加一个 maven 依赖的入口，必须写在 "allprojects" 代码块中:
在你的 app 中 build.gradle 文件中添加 React Native 依赖:

```
allprojects {
    repositories {
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        ...
    }
    ...
}
```
确保依赖路径的正确！以免在 Android Studio 运行 Gradle 同步构建时抛出 “Failed to resolve: com.facebook.react:react-native:0.x.x" 异常。
#### 2.集成Yruntime到项目
在项目中引用
```
implementation 'cn.yasin:yruntime:1.0.0'
```
#### 3.测试集成结果
##### 1. 运行 Packager

运行应用首先需要启动开发服务器（Packager）。你只需在项目根目录中执行以下命令即可：

```
$ yarn start
```
##### 2. 运行你的应用
保持 packager 的窗口运行不要关闭，然后像往常一样编译运行你的 Android 应用(在命令行中执行./gradlew installDebug或是在 Android Studio 中编译运行)。

编译执行一切顺利进行之后，在进入到 YReactActivity 时应该就能立刻从 packager 中读取 JavaScript 代码并执行和显示：

![](https://reactnative.cn/docs/assets/EmbeddedAppAndroid.png)

##### 3. 在 Android Studio 中打包
你也可以使用 Android Studio 来打 release 包！其步骤基本和原生应用一样，只是在每次编译打包之前需要先执行 js 文件的打包(即生成离线的 jsbundle 文件)。具体的 js 打包命令如下：

```
$ react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/com/your-company-name/app-package-name/src/main/assets/index.android.bundle --assets-dest android/com/your-company-name/app-package-name/src/main/res/
```

注意把上述命令中的路径替换为你实际项目的路径。如果 assets 目录不存在，则需要提前自己创建一个。
然后在 Android Studio 中正常生成 release 版本即可！



