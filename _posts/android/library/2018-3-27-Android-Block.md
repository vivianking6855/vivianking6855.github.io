---
layout: post
title: Android Block库发布
date: 2018-3-27
excerpt: "Android Block库发布"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

今天发布封装的Android Block框架

# Android 版本

Android版本：Android 4.1 API 16以上支持。

# jCenter发布地址

[AndroidBlockMonitor Github](https://github.com/vivianking6855/android-blockmonitor)

[AndroidBlockMonitor jCenter](https://bintray.com/vivianwayne1985/maven/AndroidBlockMonitor)

# 原理

基于[Choreographer.FrameCallback ](http://vivianking6855.github.io/2018/03/05/Android-optimization-6-Block/)。Choreographer类的FrameCallback是系统每隔16ms发出VSYNC信号，来通知界面进行重绘、渲染的回调。

- 利用两次回调的时间差十分超出16ms来判断卡顿
- 当每一帧被渲染时会触发回调FrameCallback， FrameCallback回调void doFrame (long frameTimeNanos)函数
- 检查两次doFrame的间隔，如果大于16.6ms说明发生了卡顿

默认卡顿阈值：80ms = 5*16.6ms

默认抓取log频率：即两次Frame刷新之间获取 次

流程图

![](https://i.imgur.com/R3DUCcr.jpg)

# 使用
   
   Gradle

    compile 'com.learn.blockmonitor:AndroidBlockMonitor:1.0.180326'

   Maven

    <dependency>
      <groupId>com.learn.blockmonitor</groupId>
      <artifactId>AndroidBlockMonitor</artifactId>
      <version>1.0.180326</version>
      <type>pom</type>
    </dependency>
    
    
1. 加载和启动Application的OnCreate

        @Override
        public void onCreate() {
            super.onCreate();
            BlockMonitor.getInstance().install(this).start();
        }

2. 暂停

    BlockMonitor.getInstance().install(this).stop();

3. 卸载
    
    BlockMonitor.getInstance().install(this).uninstall();


# 效果

---

Notification

   ![](https://i.imgur.com/HAFM9qr.jpg)

---

Log详情
    
   ![](https://i.imgur.com/Anob1Tk.jpg)
   
   ![](https://i.imgur.com/nPYwL3j.png)

空白部分后期会显示波形图（Frame-Time，Frame-DropCount等）

# log信息

- 主线程StackTrace
- Frame刷新时间，droppedCount(diff/16.6)
- CPU状态：CPU Core，是否busy等
- Memory状态：all memory, free memory等
- 外部传入备注信息（例如，package name, version, etc）


# TODO

详情上方的波形图
