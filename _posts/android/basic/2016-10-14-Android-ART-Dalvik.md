---
layout: post
title: Android 之 Dalvik和ART
date: 2016-10-14
excerpt: "Android 之 Dalvik和ART"
categories: Android
tags: [Android 基础]
comments: true
---

# 什么是Dalvik

Dalvik是Google公司自己设计用于Android平台的Java虚拟机。

Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一。

它可以支持已转换为 .dex（即Dalvik Executable）格式的Java应用程序的运行，.dex格式是专为Dalvik设计的一种压缩格式，

适合内存和处理器速度有限的系统。Dalvik 经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik 应用作为一个独立的Linux 进程执行。

独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

​Dalivk中，使用dexopt将dex文件优化为Odex文件。再依靠一个Just-In-Time (JIT)编译器去解释字节码。

# 什么是ART

Android操作系统已经成熟，Google的Android团队开始将注意力转向一些底层组件，其中之一是负责应用程序运行的Dalvik运行时。

Google开发者已经花了两年时间开发更快执行效率更高更省电的替代ART运行时。 

ART代表Android Runtime，其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time (JIT)编译器去解释字节码。

开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运 行。

ART则完全改变了这套做法，在应用安装时就预编译字节码到机器语言，这一机制叫Ahead-Of-Time (AOT）编译。

在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

ART会使用dex2aot直接将全部的dex文件编译为native code存储，

## ART优点

1. 系统性能的显著提升。
2. 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
3. 更长的电池续航能力。
4. 支持更低的硬件。

## ART缺点

1. 更大的存储空间占用，可能会增加10%-20%
2. 更长的应用安装时间。

总的来说ART的功效就是“空间换时间”。

4.4的手机打开“开发者模式”，会发现有一个选项“选择运行时环境”。可选项有Dalvik和ART（默认是Dalvik）。 

我们知道Dalvik即Android一直以来使用的运行方式。 但是从Android 4.X开始Dalvik和ART共存，用户可以选择ART。

5.X和6.0上已经删除Dalvik，只保留了ART

# 比较

| 虚拟机名字 | Android版本 | 编译 | 特性 | 
| ------------- |:-------------:| -----:| -----:| 
| Dalivk | Android 4.X及以前 | dexopt将dex文件优化为Odex文件，JIT去解析 | 
| ART | Android 4.X共存，5.X和6.0只保留了ART， | dex2aot将dex文件编译为native code存储 | 

# Reference

[Dalvik类加载机制](https://www.jianshu.com/p/1d5cac5fe699) 
