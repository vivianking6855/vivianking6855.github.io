---
layout: post
title: 性能优化（一）：方法概述
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

# 一、 布局优化

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

2. Hierarchy Viewer工具优化布局
   

详见：[性能优化（四）Google典范之Render实践中Layout优化章节](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 二、 绘制优化

1. 移除不必要的background
2. 用clipRect优化
3. onDraw方法要避免执行大量的操作

详见：[性能优化（四）Google典范之Render实践中Overdraw章节](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 三、 内存泄漏优化

内存泄漏是在开发过程需要重视的问题。内存泄漏优化分两个方面：

- 开发过程中避免写出有内存泄漏的代码
- 大量使用类似Material Design的风格，不仅安装包可以变小，还可以减少内存的占用，渲染性能与加载性能都会有一定的提升。
- 通过一些分析工具如Android Studio自动Android Monitor，[leakcanary](https://github.com/square/leakcanary)，FindBugs等

详见[Android性能优化 （二）内存 OOM](http://vivianking6855.github.io/tag/#Android%20%E8%BF%9B%E9%98%B6-ref)

内存优化并不就是说程序占用的内存越少就越好。如果因为想要保持更低的内存占用，而频繁触发执行gc操作，在某种程度上反而会导致应用性能整体有所下降。需要综合考虑权衡。

## 常用工具

- 使用[Square Leakcanary 内存泄漏](https://github.com/square/leakcanary)来排查内存泄漏
- 使用Android Lint工具（AndroidStudio内嵌工具)在开发时对code质量做监控
- FindBugs在开发时对code质量做监控
- Android Monitor实时监控

# 四、 响应速度优化

响应速度优化核心思想是避免在主线程中做耗时的操作。耗时的操作应该放到异步任务中执行。

## 常用工具

- 使用[BlockCanary 卡顿检测工具](https://github.com/markzhai/AndroidPerformanceMonitor)来检测相应速度

# 五、 ListView优化

构造Adapter时，没有使用缓存的 convertView，可以使用Android最新组件RecyclerView,替代ListView来避免

# 六、 Bitmap优化

主要是BitmapFactory.Options 和缓存。详见[Android Bitmap的加载和Cache](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

# 七、 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread. 控制最大并发数等。详见下面两篇blog~

[Android的线程和线程池，线程同步](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

可以使用RXAndroid，RXJava等并发工具

# 八、 耗电优化

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

# 九、 界面卡顿（ANR）

出现ANR的原因： 

- 主线程在5秒内没有响应输入事件 
- BroadcastReceiver在10秒内没有执行完毕

因此需要Application中药注意：

- UI线程只做界面刷新，不做任何耗时操作，耗时操作放在子线程来做 
- 可以使用Thread+handle，AsyncTask，RxAndroid/RxJava等进行逻辑处理

# 性能优化要点


# Reference

> 性能优化专题

[性能优化（一）：方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)

[性能优化（二）内存管理 & Memory Leak & OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)

[性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)

[性能优化（四）Google典范之Render实践](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

> 《Android开发艺术探索》

> [Android内存优化之OOM](http://www.csdn.net/article/2015-09-18/2825737/5)

> [Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

> [优达学城 Android 系统性能 Video](https://cn.udacity.com/course/android-performance--ud825)

> [Google Android Performance Patterns YouTube ](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

> [性能优化系列总篇](http://www.trinea.cn/android/database-performance/)

> [Android性能优化全方面解析](http://blog.csdn.net/sw950729/article/details/72124008)