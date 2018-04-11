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

对于手机程序，网络操作相对来说是比较耗电的行为。优化网络操作能够显著节约电量的消耗。

# 网络优化

网络操作会直接影响到耗电情况。

优化网络操作能够显著节约电量的消耗。

- 手机硬件各个模块中，移动蜂窝模块对电量消耗是比较大的
- 另外蜂窝模块在不同工作强度下，对电量的消耗也是有差异的。

当程序想要执行某个网络请求之前，需要先唤醒设备，然后发送数据请求，之后等待返回数据，最后才慢慢进入休眠状态。这个流程如下图所示：

![](http://img.ptcms.csdn.net/article/201504/29/554078862a5a1.jpg)

在上面那个流程中，蜂窝模块的电量消耗差异如下图所示：

![](http://img.ptcms.csdn.net/article/201504/29/5540789f4dd9f.jpg)

从图示中可以看到，激活瞬间，发送数据的瞬间，接收数据的瞬间都有很大的电量消耗，所以，我们应该从如何传递网络数据以及何时发起网络请求这两个方面来着手优化。

主要是连接服务器优化，获取数据优化等

一些建议可以参看：[移动端网络优化](http://www.trinea.cn/android/mobile-performance-optimization/)


# 小结

- 可以通过缓存 + 增量更新等来解决


# Reference

[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[Android性能优化典范（二）](https://www.csdn.net/article/2015-04-29/2824583-android-performance-patterns-season-2)







