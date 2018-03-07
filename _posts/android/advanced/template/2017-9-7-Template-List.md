---
layout: post
title: Android项目模板 2 - List加载和通讯
date: 2018-2-27
excerpt: "Android项目模板"
categories: Android
tags: [Android 进阶]
comments: true
---


# 简介

Code Template List主要是List加载和通讯，主要包含下面的控件和模块：

来自Basic模板

- BottomNavigation: 底部栏导航
- ViewPager: 滑动页面切换
- BaseActivity: 重构Base类
- Lambda插件：gradle-retrolambda
- RxJava, RxAndroid 并发和通讯

List模板新增  

- [RecyclerView](http://vivianking6855.github.io/2016/12/29/Android-RecyclerView/): 列表加载
- Apache Commons Lang: Provides extra functionality for classes in java.lang
    - [homepage](https://commons.apache.org/proper/commons-lang).
    - [JavaDoc](https://commons.apache.org/proper/commons-lang/javadocs/api-release)  
- EventBus
- 懒加载：Fragment的UI界面对用户可见的时候才加载数据


# 效果图

![](https://i.imgur.com/sL73hvO.png)

# 功能描述

- 底部bottom bar点击或滑动时viewpager切换页面
- recyclerView加载列表
- 各模块通过RxJava和RxAndroid实现通讯和并发
- 消息通讯使用EventBus

# [Github Code](https://github.com/vivianking6855/android-advanced/tree/master/Template)

# 框架

采用MVP框架

## 目录结构

- activity
- base
- fragment
- listener
- model
- presenter
- ui
- utils

## 框架说明

- MainActivity: HomePagerAdapter 加载四个Fragment
- FirstFragment
    - model: FirstDataApi 和 FirstModel处理数据和逻辑
    - presenter: FirstPresenter 响应事件和通知view更新
    - view: FirstFragment UI显示
- SecondFragment, ThirdFragment, FourthFragment
    - model: SecondDataApi 和 SecondModel处理数据和逻辑
    - presenter: SecondPresenter 响应事件和通知view更新
    - view: SecondFragment UI显示
- BaseMessageEvent: 定义了EventBus的MessageEvent可能用到的方法和状态
    - moduleTag: 多个类公用一个Event时可以通过moduleTag来区分加载数据
    - loadStatus: 数据加载的状态
    - IDataModel: 待加载的数据，接口类型，各个Model可以实现此接口
- MessageQueueUitl: 完整的展示和记录所有消息的传递流程，用于程序的逻辑功能分析


# [Android项目模板 - 目录](http://vivianking6855.github.io/2018/02/28/Template-Index/)


# Reference

- [DiffUtil](http://blog.csdn.net/zxt0601/article/details/52562770):  提高RecyclerView的差异化效率，搭配payload实现高效刷新
- [EasyRecyclerViewSidebar](https://github.com/CaMnter/EasyRecyclerViewSidebar) 侧滑栏 
- [ViewPager+TabLayout+Fragment懒加载机制完全解析]( https://www.aliyun.com/jiaocheng/22714.html)
- [Tablayout+Viewpager+Fragment组合使用以及懒加载机制](http://blog.csdn.net/qq_26936889/article/details/52680261)
- [PagerAdapter分析与Fragment懒加载的几种实现](https://www.jianshu.com/p/9eb1c379f3e7)