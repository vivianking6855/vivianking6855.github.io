---
layout: post
title: Android Proguard Retrace
date: 2018-1-23
excerpt: "Android Proguard Retrace"
categories: Android
tags: [Android 基础]
comments: true
---

# Proguard Retrace

混淆过的app出现异常时，需要对应到原始的code上

Android SDK 中有一个工具可以让我们从异常堆栈中还原ProGuard 混淆过的代码

# 工具

Windows上可以直接使用GUI界面

![](https://i.imgur.com/RqtR9r2.jpg)

也可以直接使用官方提供的在线工具 [ProGuardReTrace](https://dev-tips.org/Formatters/ProGuardReTrace)

# 使用说明

在使用时要特别注意Input的写法规则和mapping文件（mapping文件通常在：build\outputs\mapping里面）

例如：

    01-20 17:24:30.528931  2786  8975 E AndroidRuntime: java.lang.NullPointerException: Attempt to invoke virtual method 'java.lang.String com.android.contacts.g.i$a.a()' on a null object reference
    01-20 17:24:30.528931  2786  8975 E AndroidRuntime: 	at com.android.contacts.g.f.h(PrepareSmartDialDataThread.java:757)
    01-20 17:24:30.528931  2786  8975 E AndroidRuntime: 	at com.android.contacts.g.f.run(PrepareSmartDialDataThread.java:90)

这样直接写进去肯定是不行的，要改成下面的写法：

    at com.android.contacts.g.i$a.a()
    at com.android.contacts.g.f.h(PrepareSmartDialDataThread.java:757)
    at com.android.contacts.g.f.run(PrepareSmartDialDataThread.java:90)
    
语句前要加上at，而且返回值要移除（例如第一行的java.lang.String）

还有要注意mapping文件，使用Jenkins build自动生成的mapping和本地build出的mapping不同，所以要选择对应的mapping。

正常的解析效果：

![](https://i.imgur.com/fSkT7jd.jpg)


# Reference

[Proguard Retrace not working](https://stackoverflow.com/questions/10542797/proguard-retrace-not-working-with-stacktrace-runtime-info-like-e-androidruntime)