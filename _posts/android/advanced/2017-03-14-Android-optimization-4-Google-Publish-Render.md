---
layout: post
title: 性能优化（四）Google典范之Render实践
date: 2017-3-14
excerpt: "Google典范之Render实践"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

优化的思想：尽量减少布局文件的层级和降低Overdraw来减轻CPU和GPU负载。

再贴下CPU和GPU的工作，潜在的问题，检测的工具和解决方案图：
 
![](http://i.imgur.com/SiZVlJ9.png)

# 解决方案

## 1. 通过Hierarchy Viewer去检测渲染效率，去除不必要的嵌套

### (1) 删除布局中无用的控件和层级

如果找不到Hierarchy Viewer，可以看下面的“如何找到Hierarchy Viewer？”

优化前layout xml：



### (2）Layout 设计优化

除了使用Hierarchy Viewer检测无用控件和层级外，在布局设计时如果可以加入下面的优化思想就更好了。

- 有选择地使用性能较低的ViewGroup
- 采用<include>、<merge>标签和ViewStub

## 2. Overdraw方案

   通过Show GPU Overdraw去检测(开发者选项 -> 调试GPU过度绘制 -> 显示GPU过度绘制)

   最终可以通过移除不必要的背景以及使用canvas.clipRect解决大多数问题

### (1) 移除不必要的background

### (2）clipRect的妙用


# AS 三步找到 Hierarchy Viewer

看Android Studio三步找到它

## Step 1：

![](http://i.imgur.com/mCxa2Ow.jpg)

## Step 2：

![](http://i.imgur.com/6GCAp0a.jpg)

## Step 3：

![](http://i.imgur.com/5OHUC4l.jpg)

<br>


另外好需要显示下面三个窗口，不然看不到Tree View，Propertyies, Overview

![](http://i.imgur.com/vpOcJMU.jpg)

<br>

![](http://i.imgur.com/ffC691e.jpg)


# 性能优化专题

- [性能优化（一）：方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)
- [性能优化（二）内存 OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)
- [Android性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)




# Reference

<br>


> [Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

> [优达学城 Android 系统性能 Video](https://cn.udacity.com/course/android-performance--ud825)

> [Google Android Performance Patterns YouTube ](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

> [Android UI性能优化实战 识别绘制中的性能问题](http://blog.csdn.net/lmj623565791/article/details/45556391/)





