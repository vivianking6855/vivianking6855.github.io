---
layout: post
title: ReactNative 学习笔记 Code Push 热更新
date: 2016-07-15
excerpt: "Code Push 热更新"
categories: ReactNative
tags: [ReactNative]
comments: true
---

## Introduction
热更新在不用的领域有着多重多样的解释。我们关心的app的热更新简单来讲是：无需发新版本，实现app添加新功能。

这里主要介绍Microsoft退出的CodePush平台实现热更新。它是一个云端服务，支持Cordova and React Native的开发者直接更新app, 而无需重新安装apk.

## 微软的codepush

微软的codepush框架操作原理是这样：

 - （1）把react中的js打包，生成一个app（react-native官方就支持到这一步）
 - （2）把最新的js包都打包上传到微软的服务器（codepush框架实现）
 - （3）在app中判断本地js包的版本号和微软服务器的版本号，然后全部下载下来后实现更新(codepush框架实现)
 
    codepush.sync(paramas)可以实现更新，一般我们放在app启动的时候进行。或者可以在APP中增加一个按钮，点击就运行更新。
 
## 实践步骤（Windows）
可以follow[官网SOP](http://microsoft.github.io/code-push/index.html#getting_started)，也可以参考下面的步骤。

1. Install the CodePush CLI
        
    CLI用来管理account. 安装命令： 
    
        npm install -g code-push-cli
2. Create a CodePush account
   
    用微软账号或GitHub 注册CodePush账号

        code-push register
        
    [其他命令](http://microsoft.github.io/code-push/docs/cli.html#link-2)：
    - code-push login 登录
    - code-push whoami 查看登录
    - code-push logout 登出
    - code-push session ls 查看session
    - code-push session rm <machineName> 清除session
        
3. 注册your app

        注册命令：code-push app add MyApp 
        
    注册命令会产生两个key: Staging and Production，可以用"code-push deployment ls codepushdemo -k"查看
        
    MyApp是你app的名称，其他[常用命令](http://microsoft.github.io/code-push/docs/cli.html#link-3)如下：
    
    code-push app ls 列出注册的apps
    
    code-push app rm <appName> 移除注册app
    
    code-push app rename <appName> <newAppName> 重名注册app
    
    
4. app中配置configuration（以RN为例，Android）
    
   - 安装插件codepush和link

            npm install --save react-native-code-push@latest
            rnpm link react-native-code-push （需要先安装rnpm: npm i -g rnpm。命令执行时输入步骤2的key）
           
    - android/app/build.gradle 中添加codepush.gradle
           
            ...
            apply from: "../../node_modules/react-native/react.gradle"
            apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
            ...
           
    - 配置plugin-CodePush
     - MainApplication.java中添加（For React Native >= v0.29）
               
                ...
                // 1. Import the plugin class.
                import com.microsoft.codepush.react.CodePush;
            
                public class MainApplication extends Application implements ReactApplication {

                    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
                        ...
                        // 2. Override the getJSBundleFile method in order to let
                        // the CodePush runtime determine where to get the JS
                        // bundle location from on each app start
                        @Override
                        protected String getJSBundleFile() {
                            return CodePush.getJSBundleFile();
                        }
                
                        @Override
                        protected List<ReactPackage> getPackages() {
                            // 3. Instantiate an instance of the CodePush runtime and add it to the list of
                            // existing packages, specifying the right deployment key. If you don't already
                            // have it, you can run "code-push deployment ls <appName> -k" to retrieve your key.
                            return Arrays.<ReactPackage>asList(
                                new MainReactPackage(),
                                new CodePush(BuildConfig.CODEPUSH_KEY, MainApplication.this, BuildConfig.DEBUG)
                            );
                        }
                    };
                }
        - build config : build.gradle file (android/app/build.gradle）    
        
            android {
                ...
                buildTypes {
                    debug {
                        ...
                        buildConfigField "String", "CODEPUSH_KEY", '"<INSERT_STAGING_KEY>"'
                        ...
                    }
            
                    release {
                        ...
                        buildConfigField "String", "CODEPUSH_KEY", '"<INSERT_PRODUCTION_KEY>"'
                        ...
                    }
                }
                ...
            }        
            
**如果有安装rnpm，也可以用react-native link react-native-code-push 步骤4（rn > 0.27)**
                
5. app中配置升级处理code

    [sample code](https://github.com/vivianking6855/ReactNativeProject/tree/master/Advanced/codepush)

    详见：[JavaScript API Reference](https://github.com/Microsoft/react-native-code-push)
                
 
6. Release an app update （ReactNative）

    code-push release-react <appName> <platform>
    
    [其他发布命令](http://microsoft.github.io/code-push/docs/cli.html#releasing-updates-react-native)
    
## [Sample Code](https://github.com/vivianking6855/ReactNativeProject/tree/rncomponent/Examples/codepush)


## 注意事项：
1. deployment
     
    注册CodePush service, 默认会有产生两个deployment： Staging and Production. （阶段和产品） 
    
    如果需要其他部署，可以自己添加管理：[Deployment Management](http://microsoft.github.io/code-push/docs/cli.html#link-3)
    
    列出所有deployment: code-push deployment ls <appName> [--displayKeys|-k]
    





> [Microsoft React Native Client SDK](http://microsoft.github.io/code-push/docs/react-native.html)  

> [Microsoft Code Push RN Github](https://github.com/Microsoft/react-native-code-push)

> [microsoft Code Push](https://microsoft.github.io/code-push/)

> [ReactNative中文网推出的代码热更新服务](https://github.com/reactnativecn/react-native-pushy)

> [React Native热更新部署/热更新-CodePush最新集成总结](http://www.jianshu.com/p/9e3b4a133bcc)