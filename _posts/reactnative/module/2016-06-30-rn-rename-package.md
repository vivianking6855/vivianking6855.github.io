---
layout: post
title: ReactNative 学习笔记 修改包名
date: 2016-06-30
excerpt: "修改包名"
tags: [ReactNative]
comments: true
---

## Android包名修改

### 方案一
1. AndroidStudio打开项目 right click to "Open Module Setting".
    - Go to the Flavours tab. 
    - Change the applicationID to whatever package name you want. Press OK. 
2. 删除app 和 依赖包build, 例如  
    - rimraf android\app\build
    - rimraf android\build
    - rimraf node_modules\react-native-audioplayer\android\build
    - rimraf node_modules\react-native-networking\android\build
    - rimraf node_modules\react-native-rong-imlib\android\build
3. 重新打包 

    gradlew assembleRelease
4. aapt查看包名
    
    aapt dump badging "apkfile"
    
### 方案二

1. Uncheck "Compact Empty Middle Packages "
2. 修改directory为新的name
    - Right-click it
    - Select Refactor
    - Click on Rename
    - In the Pop-up dialog, click on Rename Package instead of Rename Directory

3. open Gradle Build File (app\build.gradle). Update the applicationId to your new Package Name


## IOS包名修改


<br>
<br>
<br>
> [android-studio-rename-package](http://stackoverflow.com/questions/16804093/android-studio-rename-package)

> [application-id](https://developer.android.com/studio/build/application-id.html)