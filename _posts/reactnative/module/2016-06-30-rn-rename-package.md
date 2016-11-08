---
layout: post
title: ReactNative 学习笔记 修改包名
date: 2016-06-30
excerpt: "修改包名"
tags: [ReactNative]
comments: true
---

## Android包名修改

1. AndroidStudio打开项目 right click to "Open Module Setting".
    - Go to the Flavours tab. 
    - Change the applicationID to whatever package name you want. Press OK. 
2. 调整AndroidManifest.xml中个别包名依赖项

    例如：
    
        <!-- 个推3.0电子围栏功能所需权限 -->
        <!-- 浮动通知权限 -->
        <!-- 自定义权限 -->
        <uses-permission android:name="getui.permission.GetuiService.com.wenxitek.mijiang" />
        <!--替换为第三方应用的包名-->
        <permission
                android:name="getui.permission.GetuiService.com.wenxitek.mijiang"
                android:protectionLevel="normal" >
        </permission><!--替换为第三方应用的包名-->
        <!-- 个推SDK权限配置结束 -->

        <provider
             android:name="com.igexin.download.DownloadProvider"
             android:authorities="downloads.com.wenxitek.mijiang"
             android:exported="true"
             android:process=":pushservice" />

4. 删除app 和 依赖包build, 例如  
    - rimraf android\app\build
    - rimraf android\build
    - rimraf node_modules\react-native-audioplayer\android\build
    - rimraf node_modules\react-native-networking\android\build
    - rimraf node_modules\react-native-rong-imlib\android\build
3. 重新打包 

    gradlew assembleRelease
4. aapt查看包名
    
    aapt dump badging "apkfile"



## IOS包名修改




<br>
<br>
<br>

> [android-studio-rename-package](http://stackoverflow.com/questions/16804093/android-studio-rename-package)

> [Android application-id](https://developer.android.com/studio/build/application-id.html)

> [如何在xcode中修改app的包名](http://blog.csdn.net/wolfking_2009/article/details/13986991)
