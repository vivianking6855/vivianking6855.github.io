---
layout: post
title: 优化规范 - Render
date: 2018-4-20
excerpt: "优化规范 - Render"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# 渲染优化规范

本文主要整理Android移动应用的渲染优化规范

渲染操作通常依赖于两个核心组件：CPU与GPU

- CPU负责包括Measure，Layout，Record，Execute的计算操作
- GPU负责Rasterization(栅格化)操作

CPU通常存在的问题的原因是存在非必需的视图组件，它不仅仅会带来重复的计算操作，而且还会占用额外的GPU资源。

Google优化典范上一张图可以概括：CPU和GPU的工作，潜在的问题，检测的工具和解决方案
 
![](http://i.imgur.com/SiZVlJ9.png)

我们渲染部分的优化的思想是：通过布局优化(例如尽量减少布局文件的层级等)，降低Overdraw，再搭配Lint提醒来减轻CPU和GPU负载。

- 布局优化目的是使得布局是宽而浅，而不是窄而深
- Overdraw优化目的在于减少重绘

注意自定义view无法通过工具来检测Overdraw，可以通过clipRect and quickReject来优化。同时需要注意onDraw的效率

## 1. 衡量标准

渲染的衡量标准也很难做定义，我们的标准定义做法是搭配下面思想来大约评估标准

- Hierarchy Viewer工具
- Overdraw的层级
- Lint提醒
- UI design设计最优

可以请多位专家一起来Review如何设定UI design最优的布局，Overdraw标准。

例如Contact 详情页面的标准如下：

![](https://i.imgur.com/ogo3Ie8.jpg)

不同的app可以使用场景不同，规范的准则是：尽量设计最优化布局和重绘

## 2. 分析

分析步骤如下：

1. 选择优化模块
2. 使用Hierarchy Viewer工具和Overdraw层级工具分析

    - 特别注意HV工具红色measure，layout, draw的红色点代表相对最慢，需要重点检查。
    - Overdraw最优是3层以下
    
3. 自定义View
    
    自定义view无法通过工具来检测Overdraw. 
    
    - 可以通过clipRect and quickReject来优化
    - 需要特别注意onDraw的效率

    建议：
    
    - 如无必要请使用系统自带View，系统会自动优化Overdraw。
    - 如果自定义View onDraw非常复杂，可以考虑用SurfacView. 例如波形图

## 3. 优化

常用的解决方案

1. 合理选择空间容器

    - 不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout

        RelativeLayout会让子View调用两次measure。时间和速度LinearLayout比RelativeLayout优
    
    - 如果RelativeLayout可以减少层级，请使用它。层级RelativeLayout比LinearLayout优越
    - RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin。


2.	ViewStub

    高效占位符代替View.GONE，因为设置为GONE，在Inflate布局的时候View仍然会被Inflate，即仍然会创建对象，会被实例化，会被设置属性，会耗费内存等资源。
3.	Merge

    使用Merge标签减少view层级
4.	页面背景图 
    
    - 去掉window的默认背景和其他不必要的背景
        - 如果你的layout已经设置背景，可以移除DecorView的背景。
        
                getWindow().setBackgroundDrawable(null); 
                或者在theme中添加
                android:windowbackground="null"；

    - 移除不必要背景
    
        例如父控件设置一个整体的背景，子View设置背景而且宽度mach_parent，就造成了背景重绘

    - 在布局和代码中设置背景和图片的时候，如果是纯色，尽量使用color；
    - 如果是规则图形，尽量使用shape画图；
    - 如果是复杂icon，建议使用svg格式
    - 如果svg不满足需求，建议使用9patch图；
    - 如果不能使用9patch的情况下，针对几种主流分辨率的机型进行切图，比如xxhdpi,mdpi, xxxhdpi

5.	慎用Alpha

    做Alpha转化就需要对当前View绘制两遍，绘制效率会大打折扣，耗时会翻倍。如果一定做Alpha转化的话，可以采用缓存的方式。
    
        view.setLayerType(LAYER_TYPE_HARDWARE);
        doSomeThing();
        view.setLayerType(LAYER_TYPE_NONE);
        
    通过setLayerType方式可以将当前界面缓存在GPU中，这样不需要每次绘制原始界面，但是GPU内存是相当宝贵的，所以用完要马上释放掉。
6.	自定义View优化
    
    自定义view无法通过工具来检测Overdraw ，可以通过ClipRect & QuickReject来优化重绘
    
    - canvas.clipRect()可以指定一块矩形区域，只有在这个区域内才会被绘制。

        可以很好解决重叠组件的Overdraw问题，还可以节约CPU与GPU资源。因为在clipRect区域之外的绘制指令都不会被执行，那些部分内容在矩形区域内的组件，仍然会得到绘制。
    - canvas.quickreject()
        canvas.quickreject()可以来判断是否没和某个矩形相交，从而跳过那些非矩形区域内的绘制操作。
    
    建议：
    
    - 如无必要请使用系统自带View
        
        Android系统会通过避免绘制那些完全不可见的组件来尽量减少消耗，来解决Overdraw的问题。但是复杂的自定义的View(通常重写了onDraw方法)，Android系统无法检测在onDraw里面具体会执行什么操作
        
        系统无法监控并自动优化，也就无法避免Overdraw了
        
    - 如果自定义View onDraw复杂到会导致UI卡顿，可以考虑用SurfacView. 例如波形图

# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)