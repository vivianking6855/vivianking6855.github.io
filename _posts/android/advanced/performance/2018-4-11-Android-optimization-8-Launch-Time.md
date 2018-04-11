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

响应时间指从用户操作开始到系统给用户以正确反馈的时间。

- 一般包括逻辑处理时间 + 网络传输时间 + 展现时间。
- 对于非网络类应用不包括网络传输时间。展现时间即逻辑处理和网页或 App 界面渲染时间。

响应时间是用户对性能最直接的感受。公司对app响应时间（Response Time）要求不是特别高，标准如下
    
测试工具：高速摄像机
    
测试条件：
    
 - DUT存在500筆連絡人, 每個連絡人皆有頭像 (使用test file: 500(photo).vcf)
    
测试项目：
    
- Contacts Preview Window launch  time. 联系人预览时间测试。 要求0.6s内
    - 1. Kill Contacts process (無需重開機)
    - 2. 點擊Contacts, 當手指離開螢幕時即開始計時, 當Contacts的"底圖"完成顯示即停止計時（不包含底图的文字和icon）
- Launch time of Contacts (1st) 联系人页面加载时间。 要求2s内
    - 1. Kill Phone & Contacts process (無需重開機)
    - 2. 開啟Contacts, 當手指離開螢幕時即開始計時
    - 3. 當連絡人畫面完全顯示後,停止計時
- Frame rate of scrolling contacts. 联系人滑动卡顿测试。 要求50帧内
    - 1. 開啟連絡人app
    - 2. 滑動聯絡人以載入聯絡人頭像
    - 3. 使用手指向上滑動, 計算畫面捲動的frame rate
- Frame rate of scrolling call log. 联系人滑动卡顿测试。 要求50帧内
    - 1. 開啟Phone app > 將撥號鍵收起來 (秀出整頁的call log)
    - 2. 滑動call log以載入聯絡人頭像
    - 3. 使用手指向上滑動, 計算畫面捲動的frame rate
- Response time of backing to Home。 联系人退出时间测试。要求0.8s内
    - 1.新增10筆聯絡人資料(均需有頭像、姓名、電話)
    - 2.切換到Favorite tab，選擇step1建立的10筆資料設為Favorite
    - 3.在Favorite頁面點擊Home鍵, 當手指離開Home鍵即開始計時, 至回到Home畫面

# 分析工具

启动耗时总时间监测的三种方法

1. Android 4.4之后可以在logcat中查看TAG “ActivityManager” 信息可以看到Activity启动时间

    ![](https://i.imgur.com/ybHDZCW.jpg)

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








