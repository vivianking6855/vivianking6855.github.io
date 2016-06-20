---
layout: post
title: ReactNative
date: 2015-05-25
excerpt: "ReactNative 学习笔记 5 打包和发布"
tags: [react]
comments: true
---

### 设备端Menu

手机摇晃会出现Menu

![](http://i.imgur.com/oHDFEIR.jpg)

1. ReloadJS

    重新加载JS和刷新http://localhost:8081/index.android.bundle?platform=android 同样的效果<br/>
    当应用启动运行的时候，会自动拉取这个bundle文件。<br/>
    该文件里存放的是应用的全部逻辑代码，在目录中并不存在这个文件。
    事实上，这个地址只是一个请求地址，而非真正的静态资源文件。<br/>
    <font color="red">是通过包服务器packager通过动态分析index.android.js中的依赖，并对其进行合并得到的.</font>

2. 调试

    http://localhost:8081/debugger-ui
    
3. Enable Hot Reloading

    热更新  
    
4. Enable Live Reload

    JS变动后自动刷新，不需要店家Reload JS. 不过不太稳定
    
5. 检查元素
    
    点击后可以在设备端查看各组件layout信息，给服务器端的检查元素类似
    
6. Enable Perf Monitor
    
    性能监视器，可以看到fps等信息

### 打包发布步骤

#### 1. 生成一个签名密钥
   可以用工具Eclipse，AndroidStudio生成。也可以用命令：</br>
   keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
   会生成一个叫做my-release-key.keystore的密钥库文件
   
#### 2. 将index.android.bundle下载保存到路径/android/app/src/main/assets文件夹
   文件夹不存在，可以新建。但是名字一定要是assets
   curl -k "http://localhost:8081/index.android.bundle" > android/app/src/main/assets/index.android.bundle
   这点很重要。<font color="red">如果assets目录中不存在该文件，则打包的apk在执行时显示空白。</font>
   
#### 3. 添加签名和混淆
   打开android\app中的build.gradle文件。
   加入signingConfigs用来签名（？？？内容改成自己的，如果是执行上面的命令KeyAlias不用改）</br>
   设定enableProguardInRelease为ture来开启proguard混淆。</br>
   
        signingConfigs {
            release {
                storeFile file("../../my-release-key.keystore")
                storePassword "???"
                keyAlias "my-key-alias"
                keyPassword "???"
            }
        }
    
        buildTypes {
            release {
                minifyEnabled enableProguardInReleaseBuilds
                proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
                signingConfig signingConfigs.release // add for app signkey
            }
        }
        enableProguardInReleaseBuilds = true
 
   混淆可以减小APK文件的大小。可以移除掉React Native Java（和它的依赖库中）中没有被使用到的部分。</br>
   最终有效的减少APK的大小。</br>
   如果需要添加一些库的混淆可以修改app/proguard-rules.pro文件。</br>
   
#### 4. 打包文件
   进入/android/目录，cmd执行gradle assembleRelease</br>
   打包后的文件在 android/app/build/outputs/apk目录中。例如app-release.apk（里面还有之前调试生成的app-debug开头的apk）</br>
   注意事项：</br>

   - 如果打包碰到问题可以先执行 gradle clean 清理一下。
   - 安装gradle工具（版本与android\gradle\wrapper下的一致）并配置环境变量。
   - 配置GRADLEHOME，加入%GRADLEHOME%/bin到PATH环境变量
   - cmd执行gradle -v 检查可以检查版本

#### 5. 发布apk到应用市场

![](http://i.imgur.com/S8sLXQ9.png)

> [Windows下安装使用curl命令](http://jingyan.baidu.com/article/a681b0dec4c67a3b1943467c.html)

> [gradle下载地址](http://services.gradle.org/distributions)