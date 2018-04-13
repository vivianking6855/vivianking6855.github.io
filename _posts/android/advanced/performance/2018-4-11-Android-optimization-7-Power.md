---
layout: post
title: 性能优化（七）电量优化
date: 2018-4-11
excerpt: "电量优化"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

笔者的项目暂不需要电量和网络优化，待实战后再补充具体的实战经验。这里先介绍理论上优化要点。

电量其实是目前手持设备最宝贵的资源之一，大多数设备都需要不断的充电来维持继续使用。

不幸的是，对于开发者来说，电量优化是他们最后才会考虑的的事情。但是可以确定的是，千万不能让你的应用成为消耗电量的大户。

Purdue University研究了最受欢迎的一些应用的电量消耗

- 平均只有30%左右的电量是被程序最核心的方法例如绘制图片，摆放布局等等所使用掉的
- 剩下的70%左右的电量是被上报数据，检查位置信息，定时检索后台广告信息所使用掉的

如何平衡这两者的电量消耗，就显得非常重要了。

网络操作相对来说是比较耗电的行为。优化网络操作能够显著节约电量的消耗。

# 分析工具

电量消耗的计算与统计是一件麻烦而且矛盾的事情，记录电量消耗本身也是一个费电量的事情。唯一可行的方案是使用第三方监测电量的设备，这样才能够获取到真实的电量消耗

- 手机设置选项找到对应App的电量消耗统计数据

    ![](https://i.imgur.com/xKgRdEB.jpg)
    
    ![](https://i.imgur.com/R8WwT9z.jpg)
    
    电池优化列表的app，在锁屏的时候会被系统kill
    
    ![](https://i.imgur.com/5FxydrG.jpg)

- 通过Battery Historian Tool来查看详细的电量消耗

# 网络优化

## 前言

网络操作会直接影响到耗电情况。

优化网络操作能够显著节约电量的消耗。

- 手机硬件各个模块中，移动蜂窝模块对电量消耗是比较大的
- 另外蜂窝模块在不同工作强度下，对电量的消耗也是有差异的。

当程序想要执行某个网络请求之前，需要先唤醒设备，然后发送数据请求，之后等待返回数据，最后才慢慢进入休眠状态。这个流程如下图所示：

![](http://img.ptcms.csdn.net/article/201504/29/554078862a5a1.jpg)

在上面那个流程中，蜂窝模块的电量消耗差异如下图所示：

![](http://img.ptcms.csdn.net/article/201504/29/5540789f4dd9f.jpg)

从图示中可以看到，激活瞬间，发送数据的瞬间，接收数据的瞬间都有很大的电量消耗，所以，我们应该从如何传递网络数据以及何时发起网络请求这两个方面来着手优化。

优化策略：网络请求可以通过优化请求 + 延迟 + 缓存 + 优化数据size方案来解决

- 对网络行为进行分类，区分需要立即更新数据的行为和其他可以进行延迟的更新行为，为不同的场景进行差异化处理。
    - 把非必须的操作延迟到手机网络切换到WiFi，手机处于充电状态下再执行。例如：例如上传歌曲，图片处理等，可以等到设备处于充电状态或者电量充足的时候才进行。
    - 触发网络请求的操作，每次都会保持无线信号持续一段时间，我们可以把零散的网络请求打包进行一次操作，避免过多的无线信号引起的电量消耗。
    - 关于网络请求引起无线信号的电量消耗，还可以参考[这里](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)。
- 要避免客户端对服务器的轮询操作，这样会浪费很多的电量与带宽流量。解决这个问题，我们可以使用Google Cloud Message来对更新的数据进行推送。
- 在某些必须做同步的场景下，需要避免使用固定的间隔频率来进行更新操作，我们应该在返回的数据无更新的时候，使用双倍的间隔时间来进行下一次同步。
- 通过判断当前设备的状态来决定同步的频率，例如判断设备处于休眠，运动等不同的状态设计各自不同时间间隔的同步频率。
- 请求尽可能捆绑，延迟到某个时刻统一请求
- 使用[JobScheduler](http://developer.android.com/intl/zh-cn/reference/android/app/job/JobScheduler.html)实现网络请求延迟和批量进行执行。

## 实战

1. 分析

    Android Profile监控网络数据传输

    TODO
    
2. 方案

    待实战后再补充具体的实战经验。

一些建议可以参看：[移动端网络优化](http://www.trinea.cn/android/mobile-performance-optimization/)

主要是连接服务器优化，获取数据优化等


# 小结

- 尽量减少唤醒屏幕的次数与持续的时间，使用WakeLock来处理唤醒的问题，能够正确执行唤醒操作并根据设定及时关闭操作进入睡眠状态。
- 使用WakeLock或者JobScheduler唤醒设备处理定时的任务之后，一定要及时让设备回到初始状态。

# Reference

[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[Android性能优化典范（一）](https://www.csdn.net/article/2015-01-20/2823621-android-performance-patterns/3)

[Android性能优化典范（二）](https://www.csdn.net/article/2015-04-29/2824583-android-performance-patterns-season-2)

[Android性能优化典范（三](https://www.csdn.net/article/2015-08-12/2825447-android-performance-patterns-season-3/3)

[Android性能优化典范（四）](http://geek.csdn.net/news/detail/50692)

[Google《Android性能优化》学习笔记](https://www.csdn.net/article/2015-04-15/2824477-android-performance/4)