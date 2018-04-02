---
layout: post
title: Android 精彩三方库
date: 2017-9-27
excerpt: "Android 精彩三方库"
categories: 资料集
tags: [资料集]
comments: true
lefttrees: true
---

* content
{:toc}


# 简介

在使用三方库前，建议先看这篇：[如何正确使用开源项目](https://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&mid=2650661623&idx=1&sn=ab28ac6587e8a5ef1241be7870851355#rd)

摘录要点如下：

1. 使用成熟稳定的开源项目
2. 理解原理

    对于一些框架类的开源项目，要理解其原理并且熟练掌握他的各种API，再考虑运用到公司的项目中
    
    例如如网络请求库、ORM框架、各种图片加载库、依赖注入框架等等

3. 最好不要改源码：好的开源项目一般会持续维护与更新
4. 使用Gradle远程依赖
5. 请一定要封装一层：入口统一，便于项目管理和维护
6. 做好应急，以防万一
7. 积累自己的轮子


# 图像

- Picasso: [Square公司](http://square.github.io/)
- [Fresco](https://github.com/facebook/fresco) Facebook
- [Glide](https://github.com/bumptech/glide) ： 2014年google I/O大会上发布的官方推荐
    - [Glide和Picasso对比](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0327/2650.html)

看具体需求选择库

[Picasso，Glide，Fresco比较](https://www.cnblogs.com/android-blogs/p/5786608.html)

[Android图片加载库的选择与如何封装？ ](https://www.zhihu.com/question/40028112)

# 数据

- [Gson](https://github.com/google/gson) Google： json数据解析
- [Fastjson](https://github.com/alibaba/fastjson) Alibaba： json数据解析
- [GreenDao](https://github.com/greenrobot/greenDAO) [Square公司](http://square.github.io/)： database
- [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) 数据Disk缓存
- [ASimpleCache](https://github.com/yangfuhai/ASimpleCache) android轻量级开源缓存框架

# 网络

- [Square Retrofit/Okhttp](http://vivianking6855.github.io/2017/04/08/Android-Lib-Web/) retrofit是对okhttp做了一层封装 
- [Google Volley](https://github.com/google/volley) 
- OkHttp [Square公司](http://square.github.io/)

# 并发和通讯

- [RxAndroid/RxJava](http://vivianking6855.github.io/2017/04/08/Android-Lib-Web/) RxAndroid是RxJava的扩展, 优雅地处理异步请求.
- [EventBus](http://vivianking6855.github.io/2016/12/19/Android-Lib-EventBus/) 优化的publish/subscribe消息总线， [快速使用](http://greenrobot.org/eventbus/documentation/how-to-get-started/)

# 性能检测

- [Square Leakcanary 内存泄漏](https://github.com/square/leakcanary)
- Android Lint工具，AndroidStudio内嵌工具
- [FindBugs](http://tbfungeek.github.io/2016/06/22/Android-%E8%BF%9B%E9%98%B6%E4%B9%8B%E5%B7%A5%E5%85%B7%E7%9A%84%E4%BD%BF%E7%94%A8-Findbugs/)
- [BlockCanary 卡顿检测工具](https://github.com/markzhai/AndroidPerformanceMonitor)

# 其他插件

- [retrolambda](http://vivianking6855.github.io/2017/04/10/Android-Lib-retrolambda/) Android Studio 3.0 用不到了，默认就支持了
- [Checkstyle](http://checkstyle.sourceforge.net/) 代码style检查工具

# UI

## 侧滑栏

1. [sliding DrawerLayout](https://github.com/mzule/FantasySlide)

    ![](https://raw.githubusercontent.com/mzule/FantasySlide/master/sample.gif)

2. [bubble-scroll](https://github.com/kongnanlive/bubble-scroll)

    ![](https://github.com/cdflynn/bubble-scroll/blob/master/sample/img/scroll_sample_gif.gif?raw=true)

3. [WaveSideBar](https://github.com/gjiazhe/WaveSideBar) 

    ![](https://github.com/gjiazhe/WaveSideBar/blob/master/screenshot/gif.gif?raw=true)

4. [EasyRecyclerViewSidebar](https://github.com/CaMnter/EasyRecyclerViewSidebar)

    ![](https://github.com/CaMnter/EasyRecyclerViewSidebar/raw/master/screenshots/EasyRecyclerViewSidebar.gif)

    ![](https://camo.githubusercontent.com/f8c9865581746423e0ab371c0b09304c92735bea/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f3030366c50456339677731663330763333717365646a33316179323938776d6e2e6a7067)

## 列表

1. [RecyclerViewUtils](https://github.com/captain-miao/RecyclerViewUtils)

    ![](https://raw.githubusercontent.com/captain-miao/me.github.com/master/screenshot/refresh_and_load_more.gif)


# 


# 参考资料

- [2017年Android百大框架排行榜](https://www.cnblogs.com/jincheng-yangchaofan/articles/7018780.html)
- [LearningNotes](https://github.com/francistao/LearningNotes)
- [Android 学习资料收集](https://github.com/Freelander/Android_Data)  
- [Anroid开源库 TimLiu](https://github.com/Tim9Liu9/TimLiu-Android)
- [张鸿洋 微信推送](https://github.com/hongyangAndroid/hongyangWeixinArticles)
- [Android官网博客 Android Developers Blog](http://android-developers.blogspot.com/) 
- [android学习之路 ](http://stormzhang.com/android/2014/07/07/learn-android-from-rookie/)
- [开源项目源码解析地址](http://p.codekk.com)
- [Android 开源项目分类汇总](https://github.com/Trinea/android-open-project)
- [Android库收集](https://github.com/wasabeef/awesome-android-libraries)
- [Android 开发人员不得不收集的代码](https://github.com/Blankj/AndroidUtilCode)
