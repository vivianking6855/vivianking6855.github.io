---
layout: post
title: 性能优化（八）Launch Time
date: 2018-4-11
excerpt: "Launch Time"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

# 分析工具

启动耗时总时间监测的三种方法

1. Android 4.4之后可以在logcat中看到“ActivityManager” 信息显示Activity启动时间
2. 参照从OnCreate到onWindowFocusChanged的时间差
3. 页面完全加载可以参照从OnCreate到到IdleHandler的时间差

耗时细节时间分析

1. Method Tracing：测量耗时细节
2. Systrace：在onCreate方法里面添加trace.beginSection()与trace.endSection()方法来声明需要跟踪的起止位置，系统会帮忙统计中间经历过的函数调用耗时，并输出报表。

# 常见问题场景

1. Activity的onCreate流程，过于复杂的UI的布局与渲染操作很可能导致严重的启动性能问题
2. Application的onCreate流程，耗时的初始化操作很可能导致严重的启动性能问题

# 解决方案

1. 布局耗时：
    - 可以通过布局优化解决，具体实践参看[Contact 优化 - detail页面优化](http://vivianking6855.github.io/2018/01/04/Contact-Optimization-3/)
    - 异步延迟加载：一开始只初始化最需要的布局，异步加载图片，非立即需要的组件可以做延迟加载。
2. Application的onCreate减负：区别对待初始化
    - 有些可以延迟加载
    - 有些可以放到其他的地方做初始化操作

    特别需要留意包含Disk IO操作，网络访问等严重耗时的任务，他们会严重阻塞程序的启动。

3. 自定义的启动窗口，同时可以做成品牌宣传界面或者是给用户提供一种程序已经启动的视觉效果


# Reference

[Android性能优化典范 - 第6季](http://hukai.me/android-performance-patterns-season-6/)








