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

简单来说就是一个HTTP请求工具，与volley功能类似，都是用于简化HTTP请求的库。但是相比其他工具，Retrofit的解耦更彻底：例如通过注解来配置请求参数，根据需求来选择使用不同的CallAdapter（请求适配器，如：RxJava，Java8, Guava）、Converter（反序列化工具，如json, protobuff, xml, moshi）等。

一般的小型项目okhttp基本够用。

Retrofit是在okhttp上进行封装。使用的是注解。对[restful](http://www.ruanyifeng.com/blog/2011/09/restful.html) url具有很大的优势。

- Retrofit和Java领域的ORM概念类似， ORM把结构化数据转换为Java对象，而Retrofit 把REST API返回的数据转化为Java对象方便操作
- retrofit非常适合于restful url格式的请求，更多使用注解的方式提供功能。


核心Code


    /**
     * get config sync through retrofit
     */
    public UrlConfig getConfigThroughRetrofit() {
        try {
            Retrofit retrofit = new Retrofit.Builder()
                    .baseUrl(Const.URL_CONFIG_BASE)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();

            IWebSiteService service = retrofit.create(IWebSiteService.class);
            Call<UrlConfig> call = service.getUrlConfig();
            return call.execute().body();
        } catch (Exception ex) {
            Log.w(TAG, "getConfigThroughRetrofit ex:", ex);
        }

        return null;
    }

    /**
     * search sync through retrofit
     * filter :
     * "/1.1/threadview/search?key=" + key + "&start=" + startPosition + "&limit=" + SEARCH_LIMIT + "&sortby=dateline&order=desc";
     */
    public ArrayList<SearchEntity> searchThroughRetrofit(String key) {
        try {
            final int pos = 0;
            final int limit = 5;

            Retrofit retrofit = new Retrofit.Builder()
                    .baseUrl(Const.URL_SEARCH_BASE)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();

            IWebSiteService service = retrofit.create(IWebSiteService.class);
            Call<ResponseBody> call = service.search(key, 0, 5, "dateline", "desc");

            // parse array
            JsonParser parser = new JsonParser();
            JsonArray array = parser.parse(call.execute().body().string()).getAsJsonArray();
            ArrayList<SearchEntity> list = new ArrayList<SearchEntity>();
            if (array != null) {
                for (JsonElement ele : array) {
                    SearchEntity entity = mGson.fromJson(ele, SearchEntity.class);
                    list.add(entity);
                }
            }

            return list;
        } catch (Exception ex) {
            Log.w(TAG, "searchThroughRetrofit ex:", ex);
        }

        return null;
    }
    
    
    public interface IWebSiteService {
        @GET("search")
        Call<ResponseBody> search(@Query("key") String key,
                                  @Query("start") int start,
                                  @Query("limit") int limit,
                                  @Query("sortby") String sortby, @Query("order") String order);
    
        @GET(Const.URL_CONFIG_EXTENDS)
        Call<UrlConfig> getUrlConfig();
    }         
            

### 参考文档

[Retrofit2使用简介](https://zhangjm05.coding.me/2016/07/16/retrofit2_use/)

[Retrofit2 官方Doc](http://square.github.io/retrofit/2.x/retrofit/overview-summary.html)

## 注意事项

Gson解析时，要注意对JsonArray 的解析

示例

            Response response = mOkHttpClient.newCall(request).execute();
            if (!response.isSuccessful()) {
                Log.w(TAG, "search ex:", new IOException("Unexpected code " + response));
            }

            // parse array
            JsonParser parser = new JsonParser();
            JsonArray array = parser.parse(response.body().string()).getAsJsonArray();
            ArrayList<SearchEntity> list = new ArrayList<SearchEntity>();
            if (array != null) {
                for (JsonElement ele : array) {
                    SearchEntity entity = mGson.fromJson(ele, SearchEntity.class);
                    list.add(entity);
                }
            }

            return list;



# 优雅的异步编程 RxJava & RxAndroid

## 简介

ReactiveX 是一个专注于异步编程与控制可观察数据（或者事件）流的API。

它组合了观察者模式，迭代器模式和函数式编程的优秀思想。

- 两个主要的类 ： Observable（观察者） 和 Subscriber（订阅者）
- Observable是发出数据流或者事件；Subscriber对发出的items进行处理
- 事件可以是任何东西：触摸事件，web接口调用返回的数据等

RxAndroid是RxJava在Android上的一个扩展和Retorfit、OkHttp组合起来使用，效果惊人的简洁。似乎可以完全替代eventBus和OTTO(事件总线框架浅析)。

核心Code

    public void okhttpConfigClick(View view) {
        Disposable config = Observable.fromCallable(() -> WebsiteEngine.getInstance().getConfig())
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(urlConfig -> {
                            final String result = WebsiteEngine.getGson().toJson(urlConfig.Debug).toString();
                            Log.d(TAG, "" + result);
                            mTVShow.setText(result);
                        },
                        error -> Log.w(TAG, "okhttpConfigClick ex:", error),
                        () -> Log.d(TAG, "okhttpConfigClick complete"));
        compositeDisposable.add(config);
    }

    public void okhttpSearchClick(View view) {
        final String search_key = "zenfone3";
        Disposable search = Single.fromCallable(() -> WebsiteEngine.getInstance().search(search_key))
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(list -> {
                            final String result = WebsiteEngine.getGson().toJson(list).toString();
                            Log.d(TAG, "" + result);
                            mTVShow.setText(result);
                        },
                        error -> Log.w(TAG, "okhttpSearchClick ex:", error));
        compositeDisposable.add(search);
    }
    

## 参考文档

简单的使用可以参看

[RxJava 入门](http://www.imooc.com/article/2298)

[RxJava 使用场景举例](http://blog.csdn.net/xjbclz/article/details/53151011)

更深入的使用：

[RxJava 和 RxAndroid系列介绍](http://www.cnblogs.com/zhaoyanjun/p/5175502.html)

[深入浅出RxJava系列](http://blog.csdn.net/lzyzsd/article/details/41833541)

## 内存泄漏

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

> [如何用RXJava替代EventBus进行事件的分发](http://gank.io/post/560e15be2dca930e00da1083)

> [如何用RXjava替代EventBus进行事件的分发 GitHub上的Demo](https://github.com/kaushikgopal/RxJava-Android-Samples)


