---
layout: post
title: Android 精彩三方库
date: 2017-9-27
excerpt: "Android 精彩三方库"
categories: Android
tags: [Android]
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


选择库要考虑：

1. 库是否稳定，例如作者，更新频率等
2. 库的大小
3. 库的功能是否符合要求（例如图片几级缓存等）
4. 库的架构设计是否合理（例如线程池开销）
5. 竞品优缺点对比

# 调试

1. [stetho](https://github.com/facebook/stetho)

	- 一款提供在Chrome开发者工具上调试Android app能力的开源框架
	- 作者：FaceBook
	- 官网地址： http://facebook.github.io/stetho/
	- github   https://github.com/facebook/stetho

2. logger：一款让log日志优雅显示的框架

	- 优雅的输出log信息，并且支持多种格式：线程、Json、Xml、List、Map等
	- github https://github.com/orhanobut/logger
	- 作者：Orhan Obut

# 图像

- Picasso: [Square公司](http://square.github.io/)
- [Fresco](https://github.com/facebook/fresco) Facebook
- [Glide](https://github.com/bumptech/glide) ： 2014年google I/O大会上发布的官方推荐
    - [Glide和Picasso对比](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0327/2650.html)

看具体需求选择库

[Android 三大图片加载框架的对比——ImageLoader,Picasso,Glide](https://www.cnblogs.com/fightingchendan/p/5972985.html)

[Android图片加载库的选择与如何封装？ ](https://www.zhihu.com/question/40028112)

Picasso

- [Picasso 修改缓存路径](https://blog.csdn.net/bc_2014621/article/details/54946366)
- [Android Picasso归纳以及谜一般的缓存](https://blog.csdn.net/myfwjy/article/details/52451238)
- [图片处理自带缓存的Picasso](https://blog.csdn.net/sinat_29874521/article/details/52471374)
- [源码分析: 图片加载框架Picasso源码分析](https://www.cnblogs.com/wytiger/p/6254121.html)

# 数据

- [Gson](https://github.com/google/gson) Google： json数据解析
- [Fastjson](https://github.com/alibaba/fastjson) Alibaba： json数据解析
- [GreenDao](https://github.com/greenrobot/greenDAO) [Square公司](http://square.github.io/)： database
- [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) 数据Disk缓存
- [ASimpleCache](https://github.com/yangfuhai/ASimpleCache) android轻量级开源缓存框架

# 网络

- [Square Retrofit/Okhttp](http://vivianking6855.github.io/2017/04/08/Android-Lib-Web-OkHttp-RxJava/) retrofit是对okhttp做了一层封装 
- [Google Volley](https://github.com/google/volley) 
- OkHttp [Square公司](http://square.github.io/)

# 并发和通讯

- [RxAndroid/RxJava](http://vivianking6855.github.io/2017/04/08/Android-Lib-Web-OkHttp-RxJava/) RxAndroid是RxJava的扩展, 优雅地处理异步请求.
	- [RxBinding](https://github.com/JakeWharton/RxBinding)
	- RxBus
	- RxPermission
- [EventBus](http://vivianking6855.github.io/2016/12/19/Android-Lib-EventBus/) 优化的publish/subscribe消息总线， [快速使用](http://greenrobot.org/eventbus/documentation/how-to-get-started/)

# 性能检测

- [Square Leakcanary 内存泄漏](https://github.com/square/leakcanary)
- Android Lint工具，AndroidStudio内嵌工具
- [FindBugs](http://tbfungeek.github.io/2016/06/22/Android-%E8%BF%9B%E9%98%B6%E4%B9%8B%E5%B7%A5%E5%85%B7%E7%9A%84%E4%BD%BF%E7%94%A8-Findbugs/)
- [BlockCanary 卡顿检测工具](https://github.com/markzhai/AndroidPerformanceMonitor)


# 注解

1. Butterknife
	
	- [GitHub](https://github.com/JakeWharton/butterknife)
	- [使用总结](https://blog.csdn.net/donkor_/article/details/77879630)

# 其他插件

- [Checkstyle](http://checkstyle.sourceforge.net/) 代码style检查工具

# UI

## ViewPager

1. ViewPagerIndicator：一款基于ViewPager的页面指示器开源框架

	- 官网地址 http://viewpagerindicator.com/
	- github  https://github.com/JakeWharton/ViewPagerIndicator
	- 作者：JakeWharton 

## 刷新

1. Android-PullToRefresh：一款为普通视图提供刷新UI的视图框架

	- 8.2K的star数量使它位居刷新类UI框架榜首，强大的兼容能力，该框架支持ListView，GrdiView，WebViewScrollView，ViewPager等众多View增加刷新的能力，如果你有增加上拉加载，下拉加载的需求，你应该考虑它了！
	- github https://github.com/chrisbanes/Android-PullToRefresh
	- 作者：Chris Banes

## RecyclerView

1. BaseRecyclerViewAdapterHelper：Recyvlerview通用适配器

	- 7.7K个star，github上Android 适配器排行榜第一
	- 官网地址：http://www.recyclerview.org/
	- 作者：陈宇明以及他的小伙伴

## Animation 

1. lottie-android:一款可以在Android端快速展示Adobe Afeter Effect（AE）工具所作动画的框架

	- 动画类框架第一名，github上13.3k个star证明了他的优越性，利用json文件快速实现动画效果是它最大的便利，而这个json文件也是由Adobe提供的After Effects（AE）工具制作的，在AE中装一个Bodymovin的插件，使用这个插件最终将动画效果生成json文件，这个json文件即可由LottieAnimationView解析并生成绚丽的动画效果。而且它还支持跨平台哟。
	- github  https://github.com/airbnb/lottie-android
	- 作者：Airbnb 团队

2. Material-Animations:一款提供场景转换过渡能力的动画框架

	- Android动画框架排行榜第二名，9.3k个star数量，Material-Animations提供的是场景切换的动画效果。
	- github  https://github.com/lgvalle/Material-Animations
	- 作者：Luis G. Valle

3. AndroidViewAnimations：款提供可爱动画集合的框架

	- 囊括了开发需求过程中所有的动画效果，集成进了这个简洁可爱的动画框架。7.6K的star数，仅位列lottie-android和Material-Animations两个动画框架
	- github https://github.com/daimajia/AndroidViewAnimations
	- 作者：daimajia

 
## Drawer

1. MaterialDrawer: 强大的塑料风格的抽屉框架

	- 7.6K的star数量，作者的持续更新状态，如果你还在犹豫上手SlidingMenu遇到bug没人管的困境，那么你可以入手它作为你的抽屉布局 
	- github  https://github.com/mikepenz/MaterialDrawer
	- 作者：Mike Penz

 

 33.Android-ObservableScrollView
一句话介绍：一款让视图滑动更具有视觉效果的滑动式框架

上榜理由：7.5K的star数量，证明了它曾经的价值，github上提供了12种滑动效果，你可以用它弥补其他框架的不足，提升你的App体验！

github https://github.com/ksoichiro/Android-ObservableScrollView

作者：Soichiro Kashima

使用：

compile com.github.ksoichiro:android-observablescrollview
 

34.CircleImageView
一句话介绍：圆角ImageView

上榜理由：也许你已经听说过无数种展示圆角图片的方法，但如果你不尝试尝试CircleImageView，那么你的知识库会因为少了它黯然失色，有的时候完成需求是开发者优先考虑的，不同实现方法牵扯到的性能差异更值得让人深思，如果你有心在图片性能上有所涉猎，那么CircleImageView绝对不会让你败兴而归。最后别忘了记得去看Romain Guy的建议哟。

github https://github.com/hdodenhof/CircleImageView

作者：Henning Dodenhof

使用：

dependencies {
    ...
    compile 'de.hdodenhof:circleimageview:2.1.0'
}
 

复制代码
<de.hdodenhof.circleimageview.CircleImageView
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/profile_image"
    android:layout_width="96dp"
    android:layout_height="96dp"
    android:src="@drawable/profile"
    app:civ_border_width="2dp"
    app:civ_border_color="#FF000000"/>
复制代码

# 参考资料

开源库

- [Anroid开源库 TimLiu](https://github.com/Tim9Liu9/TimLiu-Android)
- [2017年Android百大框架排行榜](https://www.cnblogs.com/jincheng-yangchaofan/articles/7018780.html)
- [Android库收集](https://github.com/wasabeef/awesome-android-libraries)
- [Android 开发人员不得不收集的代码](https://github.com/Blankj/AndroidUtilCode)

学习资料

- [LearningNotes](https://github.com/francistao/LearningNotes)
- [Android 学习资料收集](https://github.com/Freelander/Android_Data)  
- [Android官网博客 Android Developers Blog](http://android-developers.blogspot.com/) 
- [android学习之路 ](http://stormzhang.com/android/2014/07/07/learn-android-from-rookie/)
- [开源项目源码解析地址](http://p.codekk.com)
- [Android 开源项目分类汇总](https://github.com/Trinea/android-open-project)

