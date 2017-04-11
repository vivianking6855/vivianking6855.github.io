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

### 参考文档

[官方使用简介](http://square.github.io/okhttp/#overview)

[官方基本功能介绍wiki](https://github.com/square/okhttp/wiki)

[基本功能完全解析](http://blog.csdn.net/oyangyujun/article/details/46761583)这篇是中文介绍一些基本功能：

- 一般的get/post请求
- 基于Http的文件上传
- 文件下载
- 加载图片
- 消息callback，支持请求回调，直接返回对象、对象集合
- 支持session的保持

### 核心Code

    /**
     * call okhttp sync method
     */
    public UrlConfig getConfig() {
        try {
            Request request = new Request.Builder()
                    .url(Const.URL_CONFIG)
                    .build();

            Response response = mOkHttpClient.newCall(request).execute();
            if (!response.isSuccessful()) {
                Log.w(TAG, "getConfig failed " + response);
                return null;
            }

            UrlConfig config = mGson.fromJson(response.body().charStream(), UrlConfig.class);
            return config;
        } catch (Exception ex) {
            Log.w(TAG, "getConfig ex:", ex);
        }

        return null;
    }

## 2. Retrofit

### 简介

一般的小型项目okhttp基本够用。

Retrofit是在okhttp上进行封装。使用的是注解。对[restful](http://www.ruanyifeng.com/blog/2011/09/restful.html) url具有很大的优势。

- Retrofit和Java领域的ORM概念类似， ORM把结构化数据转换为Java对象，而Retrofit 把REST API返回的数据转化为Java对象方便操作
- retrofit非常适合于restful url格式的请求，更多使用注解的方式提供功能。

### 参考文档

# 优雅的异步编程

## 1. RxJava

### 简介

ReactiveX 是一个专注于异步编程与控制可观察数据（或者事件）流的API。它组合了观察者模式，迭代器模式和函数式编程的优秀思想。

RxJava 是 ReactiveX 在 Java 上的开源的实现。

各种优雅的异步线程掌控和自由的线程切换。非常的强大。

两个主要的类 ： Observable（观察者） 和 Subscriber（订阅者）

- Observable：一个 Observable是发出数据流或者事件的类
- Subscriber： 一个对这些发出的 items （数据流或者事件）进行处理（采取行动）的类
- 一个 Observable 的标准流发出一个或多个 item，然后成功完成或者出错。
- 一个 Observable 可以有多个 Subscribers
- 事件可以是任何东西：触摸事件，web接口调用返回的数据等

总之

- Observable和Subscriber可以做任何事情
    - Observable可以是一个数据库查询，Subscriber用来显示查询结果
    - Observable可以是屏幕上的点击事件，Subscriber用来响应点击事件
    - Observable可以是一个网络请求，Subscriber用来显示请求结果
- Observable和Subscriber是独立于中间的变换过程的
    - 在Observable和Subscriber中间可以增减任何数量的map
    - 整个系统是高度可组合的，操作数据是一个很简单的过程

核心Code

    // website
    private CompositeDisposable compositeDisposable;

    public void okhttpConfigClick(View view) {
        Disposable config = (Observable.fromCallable(() -> WebsiteEngine.getInstance().getConfig()).subscribeOn(Schedulers.io()))
                .subscribe(urlConfig -> {
                            final String result = WebsiteEngine.getGson().toJson(urlConfig.Debug).toString();
                            Log.d(TAG, "" + result);
                        },
                        error -> Log.w(TAG, "okhttpConfigClick ex:", error),
                        () -> Log.d(TAG, "okhttpConfigClick complete"));
        compositeDisposable.add(config);
    }

### 参考文档

简单的使用可以参看

[RxJava 入门](http://www.imooc.com/article/2298)

[RxJava 使用场景举例](http://blog.csdn.net/xjbclz/article/details/53151011)

更深入的使用：

[RxJava 和 RxAndroid系列介绍](http://www.cnblogs.com/zhaoyanjun/p/5175502.html)

[深入浅出RxJava系列](http://blog.csdn.net/lzyzsd/article/details/41833541)

## 2. RxAndroid

### 参考文档



## 3. 内存泄漏

使用 RxJava 不会魔术般的缓解内存泄露危机。为了防止可能的内存泄露，在Activity或Fragment的onDestroy 里，释放资源。

rxjava2 释放如下：

    private Disposable mConfigDisposable;


    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mConfigDisposable != null && !mConfigDisposable.isDisposed()) {
            mConfigDisposable.dispose();
        }
    }
        
这样将会停止通知，并允许垃圾回收机制释放对象，防止任何 RxJava 造成内存泄露。

如果你正在处理多个，可以用CompositeDisposable在同一时间进行释放

    private CompositeDisposable compositeDisposable;

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (compositeDisposable != null && !compositeDisposable.isDisposed()) {
            compositeDisposable.dispose();
        }
    }

# Reference

> [Android OkHttp完全解析](http://blog.csdn.net/lmj623565791/article/details/47911083)

> [Retrofit与okhttp之间的关系](http://blog.csdn.net/lmj623565791/article/details/51304204)

> [Retrofit2使用简介](https://zhangjm05.coding.me/2016/07/16/retrofit2_use/)

> [RxJava 入门](http://www.imooc.com/article/2298)

> [RxJava 使用场景举例](http://blog.csdn.net/xjbclz/article/details/53151011)

> [RxJava 和 RxAndroid系列介绍](http://www.cnblogs.com/zhaoyanjun/p/5175502.html)

> [深入浅出RxJava系列](http://blog.csdn.net/lzyzsd/article/details/41833541)