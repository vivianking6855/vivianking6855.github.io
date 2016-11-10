---
layout: post
title: ReactNative 学习笔记 RN android apk 打包
date: 2016-06-26
excerpt: "RN android apk 打包"
tags: [ReactNative]
comments: true
---



#### 1. 生成一个签名密钥

在android/app下面生成keystore file： 

   - 可以用工具Eclipse，AndroidStudio生成。<br>
   - 命令：keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
     
     密码可以自行设定，这里以123456为例
    
    会生成一个叫做my-release-key.keystore的密钥库文件
   
   
#### 2. 添加签名和混淆
打开android/app中的build.gradle文件和android/gradle.properties

- 加入signingConfigs用来签名

       -> android/app/build.gradle
    
           	signingConfigs {
                release {
                    storeFile file(MYAPP_RELEASE_STORE_FILE)
                    storePassword MYAPP_RELEASE_STORE_PASSWORD
                    keyAlias MYAPP_RELEASE_KEY_ALIAS
                    keyPassword MYAPP_RELEASE_KEY_PASSWORD
                }
            }
    
            buildTypes {
                release {
                    minifyEnabled enableProguardInReleaseBuilds
                    proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        			signingConfig signingConfigs.release
                }
            }
    
    
       -> android/gradle.properties
    
            MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
            MYAPP_RELEASE_KEY_ALIAS=my-key-alias
            MYAPP_RELEASE_STORE_PASSWORD=123456
            MYAPP_RELEASE_KEY_PASSWORD=123456

 
- 设定enableProguardInRelease为ture来开启proguard混淆。
    
    混淆可以减小APK文件的大小：proguard会移除掉React Native Java（和它的依赖库中）中没有被使用到的部分。<br>

    如果需要添加一些库的混淆可以修改app/proguard-rules.pro文件。<br>
   
#### 4. 打包文件
   - 进入/android/目录，cmd执行gradlew assembleRelease
   - 打包后的文件在 android/app/build/outputs/apk目录中。
     例如app-release.apk
    

注意：

- 如果gradlew assembleRelease时提示错误或没有安装 gradle 工具。可以自行下载后放到你的C:\Users\yournamne\.gradle\wrapper\dists下面
  - 把下载完成的zip包放入对应的文件夹，解压到当前目录
     - 例如提示下载gradle 2.4，你可以cancel
     - 此时已经生成路径： C:\Users\vivian\.gradle\wrapper\dists\gradle-2.4-all\6r4uqcc6ovnq6ac6s0txzcpc0
     - 把下载完成的zip包放入对应的文件夹，解压到当前目录
  - gradle-2.4-all.zip.part 重命名为 gradle-2.4-all.zip.ok. 这样rn就会认为已经下载完成
  - 重新运行gradlew assembleRelease
- gradle版本需要跟 /android/gradle/wrapper/gradele-wrapper.properties 文件中的版本配置保持一致。
- gradlew clean 可以清理缓存。
   

![](http://i.imgur.com/S8sLXQ9.png)



> [RN官网打包发布步骤](https://facebook.github.io/react-native/docs/signed-apk-android.html)