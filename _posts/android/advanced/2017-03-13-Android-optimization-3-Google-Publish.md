---
layout: post
title: Android性能优化（三）Google典范之开篇
date: 2017-3-13
excerpt: "Google典范之开篇"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

2015年初google发布了[Android性能优化典范](https://www.youtube.com/watch?v=qk5F6Bxqhr4&list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)，帮助开发者创建更快更优秀的Android App。

课程专题不仅仅介绍了Android系统中有关性能问题的底层工作原理

同时也介绍了如何通过工具来找出性能问题以及提升性能的建议。

主要从三个方面展开，Android的渲染机制，内存与GC，电量优化。

# 一、 Android渲染机制

首先了解下Android的渲染机制。

- Android系统每隔16ms发出VSYNC信号，触发对UI进行渲染。
- 如果每次渲染都成功，这样就能够达到流畅的画面所需要的60fps
- 这意味着程序的大多数操作都必须在16ms = 1000/60hz 内完成

![](http://i.imgur.com/0GCKaDk.png)

- 如果某个操作花费时间>16ms，系统在得到VSYNC信号的时候就无法进行正常渲染。
- 这样就发生了丢帧现象。那么用户在32ms内看到的会是同一帧画面。

![](http://i.imgur.com/lSqISqJ.png)


# 二、 卡顿的原因

大多数用户感知到的卡顿等性能问题的最主要根源都是因为渲染性能。

用户容易在UI执行动画或者滑动ListView的时候感知到卡顿不流畅，是因为这里的操作相对复杂，容易发生丢帧的现象，从而感觉卡顿。

有很多原因可以导致丢帧：没有必要的layouts、invalidations、Overdraw

- 也许是因为你的layout太过复杂，无法在16ms内完成渲染
- 有可能是因为你的UI上有层叠太多的绘制单元
- 还有可能是因为动画执行的次数过多

这些都会导致CPU或者GPU负载过重，某个操作无法在16ms内完成，造成卡顿。 Overdraw 和 VSYNC 的概念可以参看[Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

图中展示了CPU和GPU的工作，以及潜在的问题，检测的工具和解决方案

![](http://i.imgur.com/SiZVlJ9.png)


# 三、 性能问题点

## 1. UI组件Overdraw问题

引起性能问题的一个很重要的方面是因为过多复杂的绘制操作。

我们可以通过工具来检测并修复标准UI组件的Overdraw问题，但是针对高度自定义的UI组件则显得有些力不从心。

- 标准UI组件：Android系统会通过避免绘制那些完全不可见的组件来尽量减少 Overdraw。
    - Nav Drawer里面不可见的View就不会被执行浪费资源。Nav Drawer从前置可见的Activity滑出之后，不会继续绘制那些在Nav Drawer里面不可见的UI组件
- 自定义的View(重写了onDraw方法):系统无法监控并自动优化，也就无法避免Overdraw了。
    - 可以通过canvas.clipRect()来帮助系统识别那些可见的区域。这个方法可以指定一块矩形区域，只有在这个区域内才会被绘制，其他的区域会被忽视。
    - 也可以使用canvas.quickreject()来判断是否没和某个矩形相交，从而跳过那些非矩形区域内的绘制操作。

因此建议只有在必要时才使用自定义View

## 2. Memory Churn

通常来说，单个的GC并不会占用太多时间，但是大量不停的GC操作则会显著占用帧间隔时间(16ms)。

如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了。

导致GC频繁执行有两个原因：

1. Memory Churn内存抖动，内存抖动是因为大量的对象被创建又在短时间内马上被释放。
2. 瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，也会触发GC。

通过Memory Monitor我们可以查看到内存的占用情况，每一次瞬间的内存降低都是因为此时发生了GC操作。

如果在短时间内发生大量的内存上涨与降低的事件，这说明很有可能这里有性能问题。

还可以通过Heap and Allocation Tracker工具来查看此时内存中分配的到底有哪些对象。

查看在短时间内，同一个栈中不断进出的相同对象。这是内存抖动的典型信号之一。

## 3. Memory Leak

内存泄漏会很容易导致严重的性能问题。使得每级Generation的内存区域可用空间变小，GC就会更容易被触发，从而引起性能问题。

寻找内存泄漏并修复这个漏洞是件很棘手的事情，你需要对代码逻辑很熟悉，然后仔细排查。

例如，你想知道程序中的某个activity退出的时候，它之前所占用的内存是否有完整的释放干净了？

- 首先你需要在activity处于前台的时候使用Heap Tool获取一份当前状态的内存快照
- 然后你需要创建一个几乎不这么占用内存的空白activity用来给前一个Activity进行跳转
- 其次在跳转到这个空白的activity的时候主动调用System.gc()方法来确保触发一个GC操作。
- 最后，如果前面这个activity的内存都有全部正确释放，那么在空白activity被启动之后的内存快照中应该不会有前面那个activity中的任何对象了。


# 四、 问题定位和调试工具

我们可以通过一些工具来定位和调试：

## 1. Show GPU view updates检测Overdraw

开发者选项 -> 选择Show GPU view updates（显示GPU视图更新），查看视图更新的操作，

最终可以通过移除不必要的背景以及使用canvas.clipRect解决大多数问题。

![](http://i.imgur.com/BJCf3ps.png)

## 2. HierarchyViewer检查layout布局

HierarchyViewer工具查找Activity中的布局是否过于复杂

使得布局尽量扁平化，移除非必需的UI组件，去除不必要的嵌套。这些操作能够减少Measure，Layout的计算时间。

## 3. Android Studio工具，观察CPU,GPU,Memory等情况

为了寻找内存的性能问题，Android Studio提供了Android Monitor工具来帮助开发者。

- Memory Monitor：查看整个app所占用的内存，以及发生GC的时刻，短时间内发生大量的GC操作是一个危险的信号。
- Allocation Tracker：使用此工具来追踪内存的分配，前面有提到过。
- Heap Tool：查看当前内存快照，便于对比分析哪些对象有可能是泄漏了的。

![](http://i.imgur.com/9Flc6Zh.jpg)

## 4. 三方工具检查内存泄漏

- FindBugs插件
- [LeakCanary](https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)


布局优化和性能优化建议等可参看上一篇[Android的性能优化方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)


> [Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

> [优达学城 Android 系统性能 Video](https://cn.udacity.com/course/android-performance--ud825)

> [Google Android Performance Patterns YouTube ](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

> [Android UI性能优化实战 识别绘制中的性能问题](http://blog.csdn.net/lmj623565791/article/details/45556391/)





