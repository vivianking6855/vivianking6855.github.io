---
layout: post
title: ReactNative 学习笔记 修改包名
date: 2016-06-30
excerpt: "修改包名"
tags: [ReactNative]
comments: true
---

## 修改Android 报名

- AndroidStudio打开项目 right click to "Open Module Setting".
    - Go to the Flavours tab. 
    - Change the applicationID to whatever package name you want. Press OK. 
- 删除app 和 依赖包build, 例如  
    - rimraf android\app\build
    - rimraf android\build
    - rimraf node_modules\react-native-audioplayer\android\build
    - rimraf node_modules\react-native-networking\android\build
    - rimraf node_modules\react-native-rong-imlib\android\build
- 重新打包 

    gradlew assembleRelease
- aapt查看包名
    
    aapt dump badging "apkfile"

> [android-studio-rename-package](http://stackoverflow.com/questions/16804093/android-studio-rename-package)

> [application-id](https://developer.android.com/studio/build/application-id.html)