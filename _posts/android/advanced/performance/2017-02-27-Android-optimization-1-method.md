---
layout: post
title: 性能优化（一）方法概述
date: 2017-2-27
excerpt: "方法概述"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

关于性能优化的问题，主要关注的有：

- 内存
- CPU
- 耗电
- 卡顿
- 渲染
- 进程存活率等

性能优化注意事项：

- 不要过早的做性能优化，app先求能用再求好用。在需求都还没完成的时候，花大量时间在优化上是本末倒置的
- 优化要用实际数据说话，建议借助测试工具进行检测
    - 网易的Emmagee
    - 腾讯的GT和APT
    - 科大讯飞的iTest
    - Google的Battery Historian

合理优化，数据量化

# 一、 UI

详见：性能优化（四）Google典范之Render实践

## 布局优化

布局优化的思想：尽量减少层级和无用的控件。 

1. Layout 设计优化

    在布局设计时，就应该考虑最优化思想。下面列出一些常用的技巧：

    - 有选择地使用性能较低的ViewGroup.比如不嵌套的情况下，用LinearLayout和FrameLayout代替RelativeLayout.
        - RelativeLayout功能比较复杂，布局过程需要花费更多的CPU时间
        - 但是如果要LinearLayout嵌套来代替RelativeLayout，还是建议用RelativeLayout。因为嵌套同样会降低程序的性能  
    - 使用include实现布局重用，避免代码重复
    - 使用merge减少布局层级结构
    - 使用ViewStub实现延时加载
    - 在TextView中使用Compound drawable，取代ImageView + TextView
    - 使用LinearLayout自带的分割线: android:divider=""

2. [工具Hierarchy Viewer](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)优化布局

## 绘制优化

Overdraw(过度绘制)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。

在多层次的UI结构里面，如果不可见的UI也在做绘制的操作，这就会导致某些像素区域被绘制了多次。这就浪费大量的CPU以及GPU资源。

在app页面要实现非常复杂的视觉效果，可能会采用非常多的层叠组件来实现，这时候就会带来过度绘制的问题。

- 在设计时：页面之间层级关系时，要尽量保持整个界面的架构统一，大背景色一致性。
- 开发时尽量用简化的结构来布局，保持界面效果的同时也要考虑界面的流畅度

通常解决Overdraw的方法有下面三个：

1. 移除不必要的background
2. 用clipRect优化
3. canvas.quickreject()

当然如果是自定义控件，还要考虑onDraw方法要避免执行大量的操作。

如果非必要不要使用自定义控件，对于那些过于复杂的自定义的View(重写了onDraw方法)，Android系统无法检测具体在onDraw里面会执行什么操作，系统无法监控并自动优化

使用[工具Show GPU Overdraw](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)可以查看Overdraw情况。

## 界面卡顿（ANR）

出现ANR的原因： 

- 主线程在5秒内没有响应输入事件 
- BroadcastReceiver在10秒内没有执行完毕
- Service中各生命周期函数执行超过20s

因此需要Application中要注意：

- UI线程只做界面刷新，不做任何耗时操作，耗时操作放在子线程来做 
- 可以使用Thread+handle，AsyncTask，RxAndroid/RxJava等进行逻辑处理

使用[BlockCanary 卡顿检测工具](https://github.com/markzhai/AndroidPerformanceMonitor)来检测卡顿

同时下面的几项也可以纳入考虑，来提示UX：

- 做好数据缓存
- 优化逻辑处理
- 优化算法

# 二、 内存

详[Android性能优化 （二）内存管理 & Memory Leak & OOM

## 内存泄漏优化

内存泄漏是在开发过程需要重视的问题。内存泄漏优化分两个方面：

- 开发过程中避免写出有内存泄漏的代码
- 大量使用类似Material Design的风格，不仅安装包可以变小，还可以减少内存的占用，渲染性能与加载性能都会有一定的提升。
- 通过一些分析工具如Android Studio自动Android Monitor，[leakcanary](https://github.com/square/leakcanary)，FindBugs等

内存优化并不就是说程序占用的内存越少就越好。如果因为想要保持更低的内存占用，而频繁触发执行gc操作，在某种程度上反而会导致应用性能整体有所下降。需要综合考虑权衡。

## 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread. 控制最大并发数等。详见下面两篇blog~

[Android的线程和线程池，线程同步](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

可以使用RXAndroid，RXJava等并发工具

# 三、 耗电优化

电量其实是目前手持设备最宝贵的资源之一，大多数设备都需要不断的充电来维持继续使用。

Purdue University研究了最受欢迎的一些应用的电量消耗：

- 平均只有30%左右的电量是被程序最核心的方法例如绘制图片，摆放布局等等所使用掉的
- 剩下的 70%左右的电量是被上报数据，检查位置信息，定时检索后台广告信息所使用掉的。

如何平衡这两者的电量消耗，就显得非常重要了。有下面一些措施能够显著减少电量的消耗：

- 我们应该尽量减少唤醒屏幕的次数与持续的时间，使用WakeLock来处理唤醒的问题，能够正确执行唤醒操作并根据设定及时关闭操作进入睡眠状态。
- 某些非必须马上执行的操作，例如上传歌曲，图片处理等，可以等到设备处于充电状态或者电量充足的时候才进行。
- 触发网络请求的操作，每次都会保持无线信号持续一段时间，我们可以把零散的网络请求打包进行一次操作，避免过多的无线信号引起的电量消耗。
    - 关于网络请求引起无线信号的电量消耗，还可以参考[这里](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)
- App有电量消耗过多的问题，我们可以使用[JobScheduler](http://hukai.me/android-training-course-in-chinese/background-jobs/scheduling/index.htm) API来对一些任务进行定时处理
    - 例如我们可以把那些任务重的操作等到手机处于充电状态，或者是连接到WiFi的时候来处理。
- 使用WakeLock或者JobScheduler唤醒设备处理定时的任务之后，一定要及时让设备回到初始状态。
- 每次唤醒无线信号进行数据传递，都会消耗很多电量，它比WiFi等操作更加的耗电，[详情](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)


我们可以通过手机设置选项找到对应App的电量消耗统计数据。我们还可以通过Android 5.0发布的Battery History Tool来查看详细的电量消耗。

请关注程序的电量消耗，用户可以通过手机的设置选项观察到那些耗电量大户，并可能决定卸载他们。所以尽量减少程序的电量消耗是非常有必要的。

[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)
