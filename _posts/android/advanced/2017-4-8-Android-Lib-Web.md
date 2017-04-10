---
layout: post
title: 开源库 - 优雅的网络和异步编程(Retrofit/Okhttp, RxAndroid/RxJava)
date: 2017-4-8
excerpt: "开源库 - 优雅的网络和异步编程(Retrofit/Okhttp, RxAndroid/RxJava)"
categories: Android
tags: [Android 进阶]
comments: true
---

# 前言

项目中网络功能不可少的网络通讯和异常编程。

如何进行优雅的网络和异步编程？

优雅的网络编程：

popular开源的[Square公司](http://square.github.io/) 出品

- [Okhttp](https://github.com/square/okhttp)： An HTTP+HTTP/2 client for Android and Java applications.
- [Retrofit](https://github.com/square/retrofit)： Type-safe HTTP client for Android and Java

优雅的异步编程：

- [RxJava](https://github.com/ReactiveX/RxJava)： Reactive Extensions for the JVM – a library for composing asynchronous and event-based programs using observable sequences for the Java VM.
- [RxAndroid](https://github.com/ReactiveX/RxAndroid)：RxJava bindings for Android


[Github Code: WebDemo](https://github.com/vivianking6855/android-advanced)

# 优雅的网络编程

## 1. OkHttp

### 简介

OkHttp OKHttp是一款高效的HTTP客户端。

- 支持SPDY, 可以合并多个到同一个主机的请求
- 使用连接池技术减少请求的延迟(如果SPDY是可用的话)，支持连接同一地址的链接共享同一个socket，通过连接池来减小响应延迟
- 使用GZIP压缩减少传输的数据量
- 缓存响应避免重复的网络请求

### API档案

[官方使用简介](http://square.github.io/okhttp/#overview)

[官方基本功能介绍wiki](https://github.com/square/okhttp/wiki)

[基本功能完全解析](http://blog.csdn.net/oyangyujun/article/details/46761583)这篇是中文介绍一些基本功能：

- 一般的get/post请求
- 基于Http的文件上传
- 文件下载
- 加载图片
- 消息callback，支持请求回调，直接返回对象、对象集合
- 支持session的保持

## 2. Retrofit

### 简介

一般的小型项目okhttp基本够用。

Retrofit是在okhttp上进行封装。使用的是注解。对[restful](http://www.ruanyifeng.com/blog/2011/09/restful.html) url具有很大的优势。

- Retrofit和Java领域的ORM概念类似， ORM把结构化数据转换为Java对象，而Retrofit 把REST API返回的数据转化为Java对象方便操作
- retrofit非常适合于restful url格式的请求，更多使用注解的方式提供功能。

### API档案

# 优雅的异步编程

## 1. RxJava

### 简介

RxJava最核心的两个东西是Observables（被观察者，事件源）和Subscribers（观察者）。

- Observables发出一系列事件
- Subscribers处理这些事件。事件可以是任何东西：触摸事件，web接口调用返回的数据等
- 一个Observable可以发出零个或者多个事件，知道结束或者出错。
- 每发出一个事件，就会调用它的Subscriber的onNext方法，最后调用Subscriber.onNext()或者Subscriber.onError()结束

Rxjava的有点像设计模式中的观察者模式。

但是RxJava中，如果一个Observerble没有任何的的Subscriber，那么这个Observable是不会发出任何事件的。

### API档案

[RxJava 和 RxAndroid](http://www.cnblogs.com/zhaoyanjun/p/5175502.html)这篇是RxJava和RxAndroid的中文介绍


## 2. RxAndroid


# Reference

> [Android OkHttp完全解析](http://blog.csdn.net/lmj623565791/article/details/47911083)

> [Retrofit与okhttp之间的关系](http://blog.csdn.net/lmj623565791/article/details/51304204)

> [Retrofit2使用简介](https://zhangjm05.coding.me/2016/07/16/retrofit2_use/)

> [RxJava 和 RxAndroid](http://www.cnblogs.com/zhaoyanjun/p/5175502.html)