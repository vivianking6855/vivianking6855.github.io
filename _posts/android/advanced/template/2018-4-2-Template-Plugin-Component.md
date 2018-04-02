---
layout: post
title: Android组件化和插件化浅谈
date: 2018-4-2
excerpt: "Android组件化和插件化浅谈"
categories: Android
tags: [Android 进阶]
comments: true
---


# 简介

业务逻辑庞大的项目后期，必须对项目进行模块化的拆分。可以通过组件化，插件化等技术来实现。

目的是提高庞大项目的可维护性，易测性，高内聚，低耦合。

# 组件化和插件化

## [插件化](http://vivianking6855.github.io/2018/03/15/Android-Plugin/)：偏向于补丁的感觉

- app包含一个宿主 + N多插件
- 插件需要依赖宿主
- 基本不可独立运行
- 轻量级的插件化，也成为热更新。目前有很多的较为成熟的平台解决方案。

需要解决dex加载，资源加载，dex间的通讯和插件中的组件生命周期等问题。

## [组件化](https://mp.weixin.qq.com/s?__biz=MzIwMzYwMTk1NA==&mid=2247486803&idx=1&sn=884fed93567022e3ac9731df6fc4660a&scene=19#wechat_redirect)：偏向于机械零件的感觉

- app包含很多组件，比如分享，二维码等
- 每个组件原则上可以独立工作

需要解决组件间的通讯，组件的单独调试，集成调试等问题

下面是一个项目的架构图，可以看到插件在主程序外，组件是主程序的组成部分。

![](https://i.imgur.com/Cji2XGh.jpg)

# 小结

因目前项目业务逻辑复杂度不高，仅在“utillib”，“appbase”，以及业务公用库，业务MVP架构方面有涉及。

组件化除了接入一些三方组件：比如二维码识别，黄页，音效库组件等，自己还没有沉淀出好的组件。

插件化也因为项目目前对时效性要求不高，目前尚未接入