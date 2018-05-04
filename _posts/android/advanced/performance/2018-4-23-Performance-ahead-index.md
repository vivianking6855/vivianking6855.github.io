---
layout: post
title: 优化基础知识
date: 2018-4-26
excerpt: "优化基础知识 - 目录"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

本文记录要做应用优化需要了解的一些基础知识

# size优化

[apk结构介绍](https://www.aliyun.com/jiaocheng/19335.html)

[使用混淆ProGuard压缩代码和资源/减少方法数量](https://blog.csdn.net/ocean20/article/details/67634130)

# 渲染优化

[性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)

[性能优化（四）Google典范之Render实践](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 内存优化

基础原理

- [Android 操作系统的内存回收机制](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-mmry-rcycl/ )
- [Android进程的内存管理分析](http://blog.csdn.net/gemmem/article/details/8920039 )
- [android dalvik heap 浅析](http://blog.csdn.net/cqupt_chen/article/details/11068129) 
- [揭秘 ART 细节 —— Garbage collection](http://www.cnblogs.com/jinkeep/p/3818180.html)

OOM & Memory Leak

- [性能优化（二）内存管理 & Memory Leak & OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)
- [性能优化（五）内存优化](http://vivianking6855.github.io/2018/03/05/Android-optimization-5-Memory/)
- [Android性能优化之如何避免Overdraw](https://www.jianshu.com/p/145fc61011cd) 
- [Android性能优化之常见的内存泄漏](http://blog.csdn.net/u010687392/article/details/49909477) 
- [Android App 内存泄露之Handler](http://blog.csdn.net/zhuanglonghai/article/details/38233069) 

小知识点

- [Android 性能优化之String篇](http://blog.csdn.net/vfush/article/details/53038437) 
- [Java 性能优化之 String 篇](https://www.ibm.com/developerworks/cn/java/j-lo-optmizestring/)
- [HashMap，ArrayMap，SparseArray源码分析及性能对比](http://www.jianshu.com/p/7b9a1b386265) 
- [Android 开发绕不过的坑：你的 Bitmap 究竟占多大内存？](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=403263974&idx=1&sn=b0315addbc47f3c38e65d9c633a12cd6&scene=21#wechat_redirect)

# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

