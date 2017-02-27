---
layout: post
title: Android性能优化 （一）Android的性能优化方法
date: 2017-2-27
excerpt: "Android的性能优化方法"
tags: [Android]
comments: true
---

##　一、 布局优化

1. 布局优化的思想：尽量减少布局文件的层级。层级少了，Android绘制的工作量少了，程序的性能自然就高了。

2. 如何进行布局优化

(1) 删除布局中无用的控件和层级 <br>
(2）有选择地使用性能较低的ViewGroup
   
   - 比如不嵌套的情况下，用LinearLayout和FrameLayout代替RelativeLayout.
   - RelativeLayout功能比较复杂，布局过程需要花费更多的CPU时间
   - 但是如果要LinearLayout嵌套来代替RelativeLayout，还是建议用RelativeLayout。因为嵌套同样会降低程序的性能  

(3) 采用<include>、<merge>标签和ViewStub

   - <include>主要用于布局重用
   
    例如：
   
       <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
            
            <include layout="@layout/constraint_around" />
        
        </LinearLayout>
   
   - <merge>一般和<include>配合使用，可以降低减少布局的层级

        例如：如果当前布局是竖直方向的LinearLayout，而且被包含的布局文件也是竖直方向的LinearLayout。
        
        那么被包含的布局就可以通过<merge>去掉移除LinearLayout
        
        <merge xmlns:android="http://schemas.android.com/apk/res/android">
        
            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/app_name" />
        
            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/app_name" />
        
        </merge>
   
   - ViewStub 按需加载，当需要时才会将ViewStub中的布局加载到内存
    - 可以通过设定setVisibility(View.VISIBLE)或inflate显示对应的layout
    
    例如 网络 offline or page invalidate
    
        <ViewStub
        android:id="@+id/stub_offline"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inflatedId="@+id/panel_offline"
        android:layout="@layout/constraint_around" />
    

## 二、 绘制优化

绘制优化是指View的onDraw方法要避免执行大量的操作

- onDraw中不要创建新的局部对象。频繁调用时，如果一瞬间产生大量的临时对象会占用过多的内存而且会导致系统更加频发的gc，降低程序的执行效率。
- onDraw中不要做耗时任务，也不能执行成千上万的循环操作，尽管每次循环都很轻量级，但是大量的循环仍然十分强占CPU的时间片，会造成View的绘制过程不流畅。
    - Google官方给出的性能优化典范中的标准，View的绘制频率保证60fps是最佳，要求每帧的绘制时间不超过16ms(16ms = 1000/60)
    - 虽然程序很难保证16ms这个时间，但是尽量降低onDraw方法的复杂度总是有效的。

## 内存泄漏优化

内存泄漏是在开发过程需要重视的问题。内存泄漏优化分两个方面：

- 开发过程中避免写出有内存泄漏的代码
- 通过一些分析工具如MAT，[leakcanary](https://github.com/square/leakcanary)，FindBugs等

详见：[ Android性能优化 （二）内存 OOM](https://github.com/vivianking6855/vivianking6855.github.io/blob/master/_posts/android/2016-08-05-Android-optimization-2-OOM.md)

## 响应速度优化

响应速度优化核心思想是避免在主线程中做耗时的操作。耗时的操作应该放到异步任务中执行。


## ListView优化 和 Bitmap优化



## 线程优化

## 性能优化建议

