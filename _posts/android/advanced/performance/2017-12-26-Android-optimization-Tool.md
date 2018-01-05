---
layout: post
title: 性能优化 - 工具
date: 2017-12-26
excerpt: "性能优化 - 工具"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

性能优化的常用工具有很多，以下列出一些常用的工具

# 一、 [Show GPU view updates检测Overdraw （GPU）](https://developer.android.com/studio/profile/inspect-gpu-rendering.html)

开发者选项 -> 选择Show GPU view updates（显示GPU视图更新），查看视图更新的操作

最终可以通过移除不必要的背景以及使用canvas.clipRect解决大多数问题。

![](http://i.imgur.com/BJCf3ps.png)

再贴下CPU和GPU的工作，潜在的问题，检测的工具和解决方案图：
 
![](http://i.imgur.com/SiZVlJ9.png)

# 二、 [GPU呈现模式分析 （UI渲染效率）](https://developer.android.com/studio/profile/inspect-gpu-rendering.html )

开发者选项  ->  开启GPU呈现模式分析 

GPU呈现模式是一个方便快速观察UI渲染效率的工具，主要作用是实时查看每一帧的渲染效率，定位哪里存在渲染的性能问题

随着界面的刷新，界面上会滚动显示柱状图表示每帧画面说需要的渲染时间，柱状图越高表示花费的渲染时间越长。

中间有一根绿色的横线，代表每帧的最长渲染时间：16ms，需要确保每一帧花费的总时间都低于这条横线，这样才能够避免出现卡顿的问题。

从Android 6.0开始，你看到的每条柱状线已不止三种颜色：

![](https://i.imgur.com/XxxBDRR.jpg)

- Swap Buffers：表示处理任务的时间，也可以说是CPU等待GPU完成任务的时间，线条越高，表示GPU做的事情越多；
- Command Issue：表示执行任务的时间，这部分主要是Android进行2D渲染显示列表的时间，为了将内容绘制到屏幕上，Android需要使用Open GL ES的API接口来绘制显示列表，红色线条越高表示需要绘制的视图更多；
- Sync & Upload：表示的是准备当前界面上有待绘制的图片所耗费的时间，为了减少该段区域的执行时间，我们可以减少屏幕上的图片数量或者是缩小图片的大小；
- Draw：表示测量和绘制视图列表所需要的时间，蓝色线条越高表示每一帧需要更新很多视图，或者View的onDraw方法中做了耗时操作；
- Measure/Layout：表示布局的onMeasure与onLayout所花费的时间，一旦时间过长，就需要仔细检查自己的布局是不是存在严重的性能问题；
- Animation：表示计算执行动画所需要花费的时间，包含的动画有ObjectAnimator，ViewPropertyAnimator，Transition等等。一旦这里的执行时间过长，就需要检查是不是使用了非官方的动画工具或者是检查动画执行的过程中是不是触发了读写操作等等；
- Input Handling：表示系统处理输入事件所耗费的时间，粗略等于对事件处理方法所执行的时间。一旦执行时间过长，意味着在处理用户的输入事件的地方执行了复杂的操作；
- Misc Time/Vsync Delay：表示在主线程执行了太多的任务，导致UI渲染跟不上vSync的信号而出现掉帧的情况；

# 三、 HierarchyViewer检查layout布局 （CPU）

HierarchyViewer工具查找Activity中的布局是否过于复杂

使得布局尽量扁平化，移除非必需的UI组件，去除不必要的嵌套。这些操作能够减少Measure，Layout的计算时间。

Hierarchy Viewer 分析图示：

![](https://i.imgur.com/3SQWGC6.jpg)

![](https://i.imgur.com/IPbjiZa.jpg)

点击LinearLayout，然后点击三色Obtain layout time icon. 三个点从左起依次代表View的Measure, Layout和Draw的性能. 

另外颜色表示该View的该项时间指数, 分为: 

- 绿色, 表示该View的此项性能比该View Tree中超过50%的View都要快. 
- 黄色, 表示该View的此项性能比该View Tree中超过50%的View都要慢. 
- 红色, 表示该View的此项性能是View Tree中最慢的.
    - 红色说明这个View相对其他的View，该操作运行最慢，注意只是相对别的View，并不是说就一定很慢。
    - 红色的指示能给你一个判断的依据，具体慢不慢还是需要你自己去判断的。

如果你的界面的Tree View中红点较多, 那就需要注意了. 一般来说: 

- Measure红点, 可能是布局中嵌套RelativeLayout, 或是嵌套LinearLayout都使用了weight属性. 
- Layout红点, 可能是布局层级太深. 
- Draw红点, 可能是自定义View的绘制有问题, 复杂计算等.

## AS 三步找到 Hierarchy Viewer

看Android Studio如何三步找到它

### Step 1：

![](http://i.imgur.com/mCxa2Ow.jpg)

### Step 2：

![](http://i.imgur.com/6GCAp0a.jpg)

### Step 3：

![](http://i.imgur.com/5OHUC4l.jpg)

<br>


另外好需要显示下面三个窗口，不然看不到Tree View，Propertyies, Overview

![](http://i.imgur.com/vpOcJMU.jpg)

<br>

![](http://i.imgur.com/ffC691e.jpg)


# 四、 Android Studio工具，观察CPU,GPU,Memory等情况

为了寻找内存的性能问题，Android Studio提供了Android Monitor工具来帮助开发者。

- Memory Monitor：查看整个app所占用的内存，以及发生GC的时刻，短时间内发生大量的GC操作是一个危险的信号。
- Allocation Tracker：使用此工具来追踪内存的分配，前面有提到过。
- Heap Tool：查看当前内存快照，便于对比分析哪些对象有可能是泄漏了的。

![](http://i.imgur.com/9Flc6Zh.jpg)

# 五、 三方工具检查内存泄漏

- FindBugs插件
- [LeakCanary](https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)

# 六、 一些知名的性能测试工具

- [网易的Emmagee](https://github.com/NetEase/Emmagee)： CPU、内存、流量、电量等
- 腾讯的[GT](https://github.com/Tencent/GT)： CPU、内存、流量、电量、帧率/流畅度等等
- 腾讯的[APT](http://www.360doc.com/content/15/0511/10/5600807_469614293.shtml)：CPU，内存检测，内存构成分析
- 科大讯飞的iTest： 记录特定应用的性能消耗情况，包括cpu、内存、流量、电量等信息
- Google的Battery Historian： 电量优化

# Reference

<br>

[Android开发者选项——Gpu呈现模式分析](https://www.cnblogs.com/ldq2016/p/6667381.html)

[Android内存分析工具（一）：Memory Monitor](http://blog.csdn.net/berber78/article/details/47783585)




