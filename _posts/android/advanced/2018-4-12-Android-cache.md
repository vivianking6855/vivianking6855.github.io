---
layout: post
title: Android 缓存
date: 2018-4-12
excerpt: "Android 缓存"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# 前言

采用内存缓存和磁盘缓存，做好数据缓存可以很大程度优化执行时间

缓存主要包括

- 对象缓存：对象缓存能减少内存的分配
- IO缓存：IO缓存减少磁盘的读写次数
- 网络缓存：网络缓存减少网络传输
- DB缓存：DB缓存较少Database的访问次数

在内存、文件、数据库、网络的读写速度中，内存都是最优的。所以尽量将需要频繁访问或访问一次消耗较大的数据存储在缓存中

# Android中常使用缓存

- 线程池
- Android图片缓存，Android图片Sdcard缓存，数据预取缓存
- 消息缓存

    通过handler.obtainMessage复用之前的message，如下：
    handler.sendMessage(handler.obtainMessage(0, object));

- ListView缓存
- 网络缓存
- 数据库缓存http response，根据http头信息中的Cache-Control域确定缓存过期时间。
- 文件IO缓存：使用具有缓存策略的输入流，对文件、网络IO皆适用
    - BufferedInputStream替代InputStream
    - BufferedReader替代Reader
    - BufferedReader替代BufferedInputStream
- layout缓存
- 其他需要频繁访问或访问一次消耗较大的数据缓存


# Reference

[性能优化之Java(Android)代码优化](http://www.trinea.cn/android/java-android-performance/)









