---
layout: post
title: 性能优化（五）内存优化
date: 2018-3-5
excerpt: "内存优化"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

不管是做什么优化都要先找到root cause，然后分析，最后执行方案。

内存优化也不例外。

我们可以从下面的几个入手点来找root cause

- 代码逻辑
- 模拟用户操作，查看内存占用较高的点，分析HeapDump
- 页面退出后，分析HeapDump

分析HeapDump可以参考

- DominatorTree，确定占用内存最多的实例
- 通过 GC root辅助分析内存占用的来源
- 通过 RetainHeapSize 量化的分析内存占用

如果条件允许，建议加入bug管理平台。出现问题，及时回传到服务器（例如腾讯的Bugly平台等）

# 工具

- LeakCanary，腾讯的LeakInspector天网等工具
- 图片引用大图告警（能够自动检测出业务图片不合理使用：比如解码的图片尺寸大于显示尺寸2倍以上等问题）
- 内存触顶监控（能够检测出内存不足时占用内存较高的业务场景，并定位到相应的页面）
- Android Profile

# 内存分析

## 1. 代码逻辑检查

把各个模块的代码逻辑检查，看是否有无用的内存消耗

## 2. 内存主动检查

- 提前加载的数据，实际上还没用
- UI 相关数据，没有及时释放。比如大量的临时view，动画等
- 数据结构不合理，占用内存过多。比如大量的重复临时数据，无效数据等
- 图片占用内存过多。比如图片

## 3. bug

有些潜在的bug可能会导致内存占用过多，甚至出现OOM.

这个就需要测试，或用户回馈才能查到。

建议app做一些压力测试（比如人工压力测试，Monkey压力测试等）

## 4. 页面退出

页面完全退回后，原则上需要释放到资源

我们可以设置开发者选项开启“不保留活动”，强制GC后，分析HeapDump。

看图中位释放的资源是否都是必要的。

![](https://i.imgur.com/99Uc4gc.jpg)

![](https://i.imgur.com/0Jdm3Hy.jpg)

# HeapDump分析

使用Android Profile前，应对应用代码施加压力并尝试强制内存泄漏。在应用中引发内存泄漏的一种方式是，先让其运行一段时间，然后再检查堆。 泄漏在堆中可能逐渐汇聚到分配顶部。

不过，泄漏越小，您越需要运行更长时间的应用才能看到泄漏。还可以通过以下方式之一触发内存泄漏：

- 将设备从纵向旋转为横向，然后在不同的 Activity 状态下反复操作多次。 
    - 旋转设备经常会导致应用泄漏 Activity、Context 或 View 对象，因为系统会重新创建 Activity
    - 如果您的应用在其他地方保持对这些对象之一的引用，系统将无法对其进行垃圾回收。
- 处于不同的 Activity 状态时，在您的应用与另一个应用之间切换（导航到主屏幕，然后返回到您的应用）。

提示：还可以使用 [monkeyrunner](https://developer.android.com/tools/help/monkeyrunner_concepts.html) 测试框架执行上述步骤。 

1. dump所有的memory

    ![](https://i.imgur.com/0Jdm3Hy.jpg)

2. dump一段时间的的memory

    ![](https://i.imgur.com/cCph2XF.jpg)

    看图中allocate的memory是否必要，检查每一项的合理性。
    
    建议按照package name排序，先检查自己package的数据。如需必要可以搭配MAT工具一起分析。
    
    ![](https://i.imgur.com/t96Zm6j.jpg)

# Reference

[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[Android性能优化典范（三](https://www.csdn.net/article/2015-08-12/2825447-android-performance-patterns-season-3)

[Android优化文章精选](https://www.jianshu.com/p/525e9d555cf3)

[Android 内存优化总结&实践](http://mp.weixin.qq.com/s/uySEk1cwxRENneFsoReFyw)

[我这样减少了26.5M Java内存](http://mp.weixin.qq.com/s/ZGpGXM8wGiSr-jtrxU3ALA)

[Android 内存暴减的秘密](http://mp.weixin.qq.com/s/kYFxXvL2SrxhKATWgYpOPg)

[神奇的内存占用大户FinalizerReference](http://www.sohu.com/a/169062705_99921943)

[Android 性能优化之内存泄漏检测以及内存优化（上）](http://blog.csdn.net/self_study/article/details/61919483) 

[Android Training - 管理应用的内存](http://hukai.me/android-training-managing_your_app_memory/)

[Android自动dump hprof文件的功能实现](https://blog.csdn.net/cxq234843654/article/details/51376050)







