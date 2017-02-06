---
layout: post
title: 2017-1-24 RemoteViews
date: 2017-1-24
excerpt: "RemoteViews"
tags: [Android]
comments: true
---

## 一、RemoteViews含义

RemoteViews:远程View

- 和远程Service一样，RemoteViews表示一个View的结构，可以在其他进程中显示。
- 由于它在其他进程中显示，为了能够更新它的界面，RemoteViews提供了一组基础的操作作用于跨进程更新它的界面。
- RemoteViews在Android中的使用场景有两种
    - Notification通知栏
    - 桌面小部件


## 二、RemoteViews的基础概念

- PendingIntent
    - 待定，即将发生的Intent
    - 在将来的某个不确定的时刻发生，不同于Intent的立刻发生
    - 典型使用场景是给RemoteViews添加点击事件，因为RemoteViews是在远程使用，没办法像其他View一样添加onClickLisenter事件
    - 三种待定意图
        - 启动Activity
        - 启动Service
        - 发送广播

## 四、RemoteViews内部机制

- 应用中每调一次set方法，RemoteViews中就会添加一个对应的Action对象
- 用NotificationManager和AppWidgetManager来提交我们的更新时，Action对象就会传输到远程进程并在远程进程中依次执行
- 远程进程通过RemoteViews的apply方法来进行View的更新操作，RemoteViews的apply方法内部则会去遍历所有的Actions对象并调用

![](http://i.imgur.com/oGJR6JL.jpg)


## 五、RemoteViews实践

模拟的通知栏效果并实现跨进程的UI更新

- 有2个Activity分别运行在不同的进程中，一个A,一个B
- A模拟通知栏的效果。修改A的process属性，让它运行在单独的进程中。
- B可以不停的地发送通知栏消息（模拟消息）。通知方案选用Broadcast（系统是用Binder）

核心Code:





<br/>
<br/>


> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)
