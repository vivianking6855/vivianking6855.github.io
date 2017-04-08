---
layout: post
title: 开源库 - Retrofit & Okhttp
date: 2017-4-8
excerpt: "开源库 - Retrofit & Okhttp"
categories: Android
tags: [Android 进阶]
comments: true
---

# 前言

popular开源的[Square公司](http://square.github.io/) 出品

Retrofit： Type-safe HTTP client for Android and Java

Okhttp： An HTTP+HTTP/2 client for Android and Java applications.

# Retrofit

# OkHttp

OkHttp OKHttp是一款高效的HTTP客户端

- 支持SPDY, 可以合并多个到同一个主机的请求
- 使用连接池技术减少请求的延迟(如果SPDY是可用的话)，支持连接同一地址的链接共享同一个socket，通过连接池来减小响应延迟
- 使用GZIP压缩减少传输的数据量
- 缓存响应避免重复的网络请求

基本功能：

- 一般的get/post请求
- 基于Http的文件上传
- 文件下载
- 加载图片
- 消息callback，支持请求回调，直接返回对象、对象集合
- 支持session的保持

优点

- 支持SPDY, 可以合并多个到同一个主机的请，使用连接池技术减少请求的延迟(如果SPDY是可用的话) ，
- 使用GZIP压缩减少传输的数据量
- 缓存响应避免重复的网络请求、拦截器等等

缺点

- 第一缺点是消息回来需要切到主线程，主线程要自己去写
- 第二传入调用比较复杂。


# Reference

> [Android OkHttp完全解析](http://blog.csdn.net/lmj623565791/article/details/47911083)

> [okhttp教程](http://blog.csdn.net/oyangyujun/article/details/46761583)

> [okHttp使用及优缺点](http://blog.csdn.net/apple_hsp/article/details/50964923) 