---
layout: post
title: 优化规范 - Time
date: 2018-4-19
excerpt: "优化规范 - Time"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# Time优化规范

本文主要整理Android移动应用的Time优化规范

Response time 响应时间指从用户操作开始到系统给用户以正确反馈的时间

- 一般应用：逻辑处理时间 + 网络传输时间 + 展现时间(网页或 App 界面渲染时间
- 非网络类应用：逻辑处理时间 +展现时间

PS：展现时间是指：网页或 App 界面渲染时间


## 1. 衡量标准

在特定测试工具（比如高速摄像机）、特定环境（比如一定量的内容数据）和测试步骤条件下，

对response time的要求，例如Contact的要求

![](https://i.imgur.com/jAkscA7.png)

不同的app可以参考公司规范，竞品和优化后的时间来定义。

## 2. 分析

分析步骤如下：

1.	选择优化模块
2.	使用工具分析模块response time

    统计表中的TC1-1到TC1-4耗时是否符合

3.	若出现耗时不符合情况，再使用工具分析方法耗时详情，定位问题

    方法耗时细节，Method Tracing 和 Android Profile CPU可任选其一。

    例如Contact中有一个Activity中有查询DB的动作耗时29ms.
    
    上次面试的记错了时间以为是100多ms，-_-||
    
    （特意把名字改了下，以便观看）
    
    ![](https://i.imgur.com/kATHjiJ.jpg)


## 3. 优化

检查方法相关的逻辑性，耗时方法可以移到线程中完成（ThreadPool，AsyncTask，HandlerThread等）

以下是一些常见问题和解决方案

1.	布局耗时

    例如Activity的onCreate流程，过于复杂的UI的布局与渲染操作可能会导致的严重的启动性能问题。
    
    常用解决方案：
	- 通过布局优化，减少渲染压力
	- 异步延迟加载：仅加载必要的布局，图片等。非必要的组件建议延迟加载。

2.	耗时的初始化
    
    例如Application的onCreate的负担太重，耗时的初始化操作可能导致严重的启动性能问题
    
    常用解决方案：
    - 区别对待初始化，非必要初始化建议延迟加载或放到其他合适的时机（例如IdleHandler里面）
    - 包含Disk IO操作、DB、网络访问等严重耗时的任务，会严重阻塞程序的启动，强烈建议放在线程池，AsyncTask或HandlerThread等子线程中实现

3.	自定义的启动窗口
    
    做成品牌宣传界面或者是给用户提供一种程序已经启动的视觉效果


# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)