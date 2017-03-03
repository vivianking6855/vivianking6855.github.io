---
layout: post
title: 2016-10-10 Android 之 微技巧 （二）Android Design Support Library
date: 2016-10-10
excerpt: "外观和交互三个重要工具：Design Library, AppCompat, Support Library"
tags: [Android 微技巧]
comments: true
---

对于构建一流外观和交互的现代应用，有三个非常重要的工具：

- Design Library
- AppCompat
- 所有 Android Support Library

## Material Design 开发利器 ： Android Design Support Library

官方在Android Support Library 22.2.0中增加了Design Support Library用来在低版本中支持Material Design。

[Material Design 开发利器](http://www.open-open.com/lib/view/open1433490948291.html)

### 使用
Design Library 已经公开发布了，需要在 SDK manager 中升级 Android Support Repository。  Gradle 可以直接加入依赖：

compile 'com.android.support:design:22.2.0'

注意 ：Design Library 依赖于 Support v4 和 AppCompat Support 库，它们会自动被加进编译依赖里来。

并且，这些新的 widget 在 Android Studio Layout 编辑器中也是可用的（在 CustomView 中找到他们）。

## [ANDROID SUPPORT兼容包详解 ](http://stormzhang.com/android/2015/03/29/android-support-library/)

Android一些SDK比较分裂，为此google官方提供了Android Support Library package 系列的包来保证高版本sdk开发的向下兼容性。

你可能经常看到v4，v7，v13这些数字，首先我们就来理清楚这些数字的含义，以及它们之间的区别：

- support-v4
    - 用在API lever 4(即Android 1.6)或者更高版本之上。
    - 包含了相对更多的内容，而且用的更为广泛，例如：Fragment，NotificationCompat，LoadBroadcastManager，ViewPager，PageTabStrip，Loader，FileProvider 等
    - Gradle引用方法：compile 'com.android.support:support-v4:21.0.3'
- support-v7
    - 为了考虑API level 7(即Android 2.1)及以上版本而设计。
    - v7是要依赖v4这个包。
    - v7支持了Action Bar以及一些Theme的兼容。
    - 不包含更低，如果不考虑1.6,可以用这个包。
    - Gradle引用方法: compile 'com.android.support:appcompat-v7:21.0.3'
- support-v13
    - 为了API level 13(即Android 3.2)及更高版本。
    - 一般不常用，平板开发中能用到。

<br>
<br>


> [DesignSupportLibraryDemo](https://github.com/xuyisheng/DesignSupportLibraryDemo)

> [google Material Design](https://www.google.com/design/spec/layout/structure.html#)

> [google support-library](https://developer.android.com/topic/libraries/support-library/features.html)

> [Android Support v4、v7、v13 介绍](http://blog.csdn.net/sunpeng1117/article/details/20696425)

> [Android Design Support Library新变化（导航视图、悬浮ActionBar..）](http://www.open-open.com/lib/view/open1433473787104.html)
