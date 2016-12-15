---
layout: post
title: 2016-10-14 Android 之 微技巧 （三）
date: 2016-10-14
excerpt: "Spannable, Gradle, DiffUtil，library中的switch, AndroidUtilCode,SurfaceView"
tags: [Android]
comments: true
---


## 1. [通过Spannable对象设置textview的样式](http://www.cnblogs.com/tianzhijiexian/p/4222393.html)
 
## 2. [Android Gradle Plugin指南](http://blog.csdn.net/qinxiandiqi/article/details/37757065)
   - [Build Variants（构建变种版本）](http://blog.csdn.net/qinxiandiqi/article/details/37906449) 

## 3. [DiffUtil](http://blog.csdn.net/zxt0601/article/details/52562770) support-v7:24.2.0的新工具类

   用来比较两个数据集，寻找出旧数据集和新数据集的最小变化量。它最大的用处就是在RecyclerView刷新时，搭配payload实现高效刷新。 

   - [讲解Demo](https://github.com/mcxtzhang/DiffUtils)   
   - [效果展示](http://www.itwendao.com/article/detail/51690.html)  
   - [使用 Payload 提高 RecyclerView 渲染效率](http://www.tuicool.com/articles/EnyARvQ)

## 4. [Android library中不能使用switch-case访问资源ID](http://zmywly8866.github.io/2014/12/24/android-can-not-use-switch-to-load-resource-in-libproject-solution.html)
解决方案是：改为 if else

## 5. [AndroidUtilCode-Android 开发人员不得不收集的代码](https://github.com/Blankj/AndroidUtilCode)

## 6. [View、SurfaceView、SurfaceTexture](http://blog.csdn.net/fulinwsuafcie/article/details/48583849)
- [SurfaceView的实现原理分析](http://blog.csdn.net/luoshengyang/article/details/8661317)
- SurfaceView和View最本质的区别在于，surfaceView是在一个新起的单独线程中可以重新绘制画面。而View必须在UI的主线程中更新画面。
- 被动更新画面的。比如棋类，用view就好了。

    因为画面的更新是依赖于 onTouch 来更新，可以直接使用 invalidate。 
    
    因为这种情况下，这一次Touch和下一次的Touch需要的时间比较长些，不会产生影响。
    
- 主动更新。比如一个人在一直跑动或者波形图需要surfaceView来控制（google专门扩展用于2D游戏开发的画布）
    
    需要单独的thread不停的重绘状态，避免阻塞Main UI thread
- GLSurfaceView渲染图片的效率高于SurfaceView的30倍（Android中3D游戏专用画布）
  
    GLSurfaceView的效率主要是因为机器硬件的GPU加速，现在flash技术也有了GPU加速技术
    
- SurfaceTexture不需要显示到屏幕上。适合于需要对图像流处理后再显示。例如Android Camera开发

    可以用SurfaceTexture接收来自camera的图像流。
    
    然后从SurfaceTexture中取得图像帧的拷贝进行处理。
    
    处理完毕后再送给另一个SurfaceView用于显示即可。

游戏中场景应用小结：

- 如果你的游戏不吃CPU，用View就比较好，符合标准Android操作方式，由系统决定刷新surface的时机。
- 一般2D游戏开发使用SurfaceView足够，因为它也是google专们扩展用于2D游戏开发的画布
- 不要认为什么都要使用GLSurfaceView（openGL），而且GLSurfaceView的弊端在于适配能力差，因为很多机型中是没有GPU加速的。





<br>


> [android support-library revisions](https://developer.android.com/topic/libraries/support-library/revisions.html)