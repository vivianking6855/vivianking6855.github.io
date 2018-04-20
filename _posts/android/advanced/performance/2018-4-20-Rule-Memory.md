---
layout: post
title: 优化规范 - Memory
date: 2018-4-19
excerpt: "优化规范 - Memory"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# 内存优化规范

本文主要整理Android移动应用的内存优化规范

Random Access Memory(RAM)在任何软件开发环境中都是一个很宝贵的资源。

在物理内存很有限的移动操作系统上，显得尤为突出。

尽管Android的Dalvik虚拟机有常规的垃圾回收，但是app的内存分配与释放的时机与地点也是不容忽略的。

规范的核心是尽力避免OOM和内存泄露。

## 1. 衡量标准

内存的衡量标准很难有非常统一的标准。

Android为每一个app都设置了一个硬性的heap size限制，超过了就会出现OOM（这里Memory Leak不是考虑的主角，因为一般Leak都可以通过工具或压力测试发现）。

先来看看vm的几个关键参数（位于system/build.prop）：

- dalvik.vm.heapstartsize
    
    单个进程堆分配的初始大小。值越大系统ram消耗越快，但是程序更流畅。
    
    dalvik.vm.heapstartsize=12m （来自某款3G手机）
- dalvik.vm.heapgrowthlimit       
    
    单个进程内存最大内存（仅dalvik堆，不包括native堆）。vm heap是可增长的，但是正dvm heap的大小是不会超过dalvik.vm.heapgrowthlimit的值，如果超过该值，则将引发OOM
    
    dalvik.vm.heapgrowthlimit=256m （来自某款3G手机）
- dalvik.vm.heapsize 

    单个进程可用的最大内存，如果存在heapgrowthlimit参数，则以heapgrowthlimit为准。
    
    通过”android:largeHeap” 为 true，vm heap最大可达heapsize，即512m。

    dalvik.vm.heapsize=512m （来自某款3G手机）

如何尽可能的避免OOM是我们的标准参考的依据。

我们考虑取heapsize的2/3作为最大值。最大值的1/10最为increase的最大值。

如果你的app要在多个品牌的手机下，可以选取几款主流市场占有率较高的设备，来综合评估。

例如Contact 详情页面的标准如下：

![](https://i.imgur.com/dO3QO7f.png)

## 2. 分析

分析步骤如下：

A.	选择优化模块

B.	使用工具（Android Profile, MAT）或代码埋点dump heap方式，查看heap分配

   - 操作一段时间后，开始分析heap和动态allocation
   - 按照包名分析top 10, 1M以上的heap分配
   - Android Profile监测线程和Service创建情况

C.	退出后的memory监测，查看是否有内存没有及时释放，避免内存泄漏等问题

## 3. 优化

常见问题和常用解决方案如下：

1.	内存泄露
    - 资源使用完毕后及时关闭（Broadcast，File, Cursor, Stream, Bitmap, ContentObserver等）
    - 谨慎使用静态变量，因为它的生命周期很长
    - Activity，View等实例请使用弱引用后软引用给其他类持有
    - 匿名内部类，内部类会持有外部类的请引用
        - 请尽量使用静态内部类
        - 如果一定要使用匿名内部类，请注意不要有耗时操作
        - 内部类请注意不要有耗时的操作，或者及时回收
        
    例如Handler导致的内存泄漏，可以通过下面两种方法解决
    
    a)	Handler声明为静态类，通过弱引用持有Activity，onDestroy的时候mHandler.removeCallbacksAndMessages
    
    b)	关闭Activity的时候停掉你的后台线程。同时removeCallbacksAndMessages
    
    常见的内存泄漏请参看000 Android_1.0.0_高性能开发手册.docx
    
2.	图片优化

Android应用里，最耗费内存的就是图片资源

- 图片分辨率
    
    Res下的图片，若当前设备的分辨率对应的图片不存在，图片会被缩放显示
    
    例如一张1024×1024的图片放在xhdpi（ARGB 8888）
    
    xhdpi的设备拿到的大小是：1024×1024×4（4M）
    
    xxhpi的设备拿到的大小是：1024*3/2×1020*3/2 ×4 （9M） 
    
    相差2倍多，在移动设备来说几M的差距还是很大
    
- 图片格式

    如果用不到Alpha通道，建议使用jpg图片，用 RGB565的格式（设置inPreferredConfig）加载大小只有 ARGB8888的一半
    
    另外还要注意合理选择Bitmap的像素格式，ARGB8888是最常用的格式，但是如果是jpg，无Alpha通道RGB565格式比较理想。
    
    如果色值丰富那么用jpg，如果作为按钮的背景，请用 png
    
- inSampleSize压缩图片

    大图小用用采样，小图大用用矩阵

    使用BitmapFactory.Options设置inSampleSize压缩图片
    
    压缩尺度可以通过inJustDecodeBounds 加上view实际需要的大小计算
    
- inBitmap重复利用图片

    官方推荐使用的参数，表示重复利用图片内存，减少内存分配
    
    在4.4以前只有相同大小的图片内存区域可以复用，4.4以后只要原有的图片比将要解码的图片大既可以复用了

- Matrix矩阵小图画大

    大图小用用采样，小图大用用矩阵
    
    用矩阵绘制出来的图是放大以后的效果，不过占用的内存仍然是采样出来的大小

- 缓存通用的Bitmap对象

    如果有在同一页面多次用到同一张图片，建议缓存Bitmap对象。缓存可以避免新建多个Bitmap对象，避免内存的浪费。

    例如联系人详情页面的默认背景和用户的头像列表中的默认头像。

- 及时回收Bitmap的内存，不要等Android系统来进行释放

        public static void safeRecycle(Bitmap bitmap) {
            if (bitmap != null && !bitmap.isRecycled()) {
                bitmap.recycle();
                bitmap = null;
            }
        }

- OOM捕获异常

        try {
        　　// 实例化Bitmap
        　　bitmap = BitmapFactory.decodeFile(path);
        　　} catch (OutOfMemoryError e) {
        　　//
        　　}
        if (bitmap == null) {
        　　// 如果实例化失败 返回默认的Bitmap对象
        　　return defaultBitmap;
        }

- 简单的效果考虑自定义View代替多张图

    如果是简单的效果，比如简单的loading，可以考虑自定义View实现
    
    例如加载图片是一系列的图5张300*300 png图，还要放到多个分辨率下面

- 复用图片

    如果图片可以通过旋转其他图片得到，就无需新增一张图片
    
    Android可以动态设置SVG的颜色，可以减少相关的图片数量
    
3. 内存抖动

    内存抖动是短时间发生了多次内存的涨跌，严重可能会造成OOM。
    
    比较容易出现的场景，onDraw，列表加载，递归，for循环等

    例如：很经典的案例是string拼接创建大量小的对象

    可以通过对象在外部定义，StringBuilder代替String等方法来避免
    
4.	其他
    
    例如常用数据结构优化，不建议枚举，ListView复用，多进程等请参看编程规范中对“管理应用的内存”的详细描述

# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[Android 内存优化总结&实践](https://mp.weixin.qq.com/s/2MsEAR9pQfMr1Sfs7cPdWQ)

[Android 开发绕不过的坑：你的 Bitmap 究竟占多大内存？](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=403263974&idx=1&sn=b0315addbc47f3c38e65d9c633a12cd6&scene=21#wechat_redirect)