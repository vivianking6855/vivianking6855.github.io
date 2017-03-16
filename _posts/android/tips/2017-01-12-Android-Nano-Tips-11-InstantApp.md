---
layout: post
title: Android InstantAPP - 即搜即得应用
date: 2017-1-12
excerpt: "Android InstantAPP - 即搜即得应用"
tags: [Android 基础]
comments: true
---

## Android InstantAPP 是什么

- 不需要安装就可以使用的android 程序。
- 是真的android程序，不是H5假冒的，不是微信小程序那种哦

示例场景：

你在 Android手机上，朋友给你发来一个链接，比方说，一个摄影用品商店 B&H 上的商品。

而恰好 B&H 的 Android 应用也支持了 Instant Apps。

你点击了这个链接，就直接进入了 B&H 的 Android 应用，即便手机并没有安装它。

![](http://i.imgur.com/klQzBR0.gif)

## 核心原理

- user打开连接时，google play Service会从play store检查 对应app是否支援Instant Appp
- 如果支持，会从play store下载对应的资源（不会下载整个apk）
- 下载完成后会在device上运行

## 哪些设备可以使用

Android 即搜即得应用与运行 Android 4.1（API 级别 16）或更高版本且安装有 Google Play 服务的设备兼容。

## 开发者是否需要开发两种不同的 Android 应用？

开发者只需使用一个源树维护一个项目。开发者只需将项目配置为构建两种工件：可安装的 APK 和即时版本。

一些开发者可能只要不到一天的时间就可以准备好并运行应用，不过，涉及的工作将因应用开发方式的不同而有所差异。

## 即搜即得应用可以使用哪些 Android API 和功能？ 

Android 即搜即得应用功能是对现有 Android 应用功能的补充而非替代。

Android 即搜即得应用使用相同的 Android API、相同的项目、相同的源代码。

为了确保符合用户对无需安装即可运行的应用的期望，Android 即搜即得应用可能会限制某些不匹配的功能。

例如，即搜即得应用无法使用后台服务、发送后台通知或访问唯一设备识别符。

## Android 即搜即得应用如何管理权限？ 

Android 即搜即得应用使用 Android 6.0（API 级别 23）中引入的运行时权限模型。

## 资源

> [视频链接介绍](https://www.youtube.com/watch?v=cosqlfqrpFA)

> [开发者页面](https://developer.android.com/topic/instant-apps/index.html )

需要注册申请

## 总结

### 限制

- user需要上传app到google play
- 4.1以上才可支援
- 需要有Google play Service支援
- 功能上有显示，例如应用无法使用后台服务、发送后台通知或访问唯一设备识别符。

### 亮点

- 即搜即得，方便，省空间
- 无需重新开发：
    - 开发者只需使用一个源树维护一个项目。开发者只需将项目配置为构建两种工件：可安装的 APK 和即时版本。
    - Android 即搜即得应用功能是对现有 Android 应用功能的补充而非替代。
- 无需搭配新的库：Android 即搜即得应用，使用相同的 Android API、相同的项目、相同的源代码。