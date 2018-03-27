---
layout: post
title: Android 插件化
date: 2018-3-15
excerpt: "Android 插件化"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 插件化的出现

1. Android系统DexOpt问题

    当Android系统安装一个应用的时候，有一步是DexOpt工具对Dex优化。DexOpt的执行过程是在第一次加载Dex文件的时候执行的。
    
    这个过程会生成一个ODEX文件，即Optimised Dex。执行Odex效率会比直接执行Dex文件的效率要高很多
    
    但是在早期的Android系统中DexOpt有一个问题：存储方法id的链表的长度是short类型，导致了方法 id 的数目不能够超过65536个

2. 插件化的概念

    当一个项目足够大的时候，显然这个方法数的上限是不够的。尽管在新版本的 Android系统中，DexOpt修复了这个问题，但是我们仍然需要对低版本的 Android系统做兼容
    
    为了解决方法数超限的问题，需要将该dex文件拆成两个或多个，为此谷歌官方推出了 multidex 兼容包，配合 AndroidStudio 实现了一个 APK 包含多个 dex 的功能。
    
动态加载技术/插件化技术需要：插件dex如何加载到内存，资源加载问题，插件互相调，四大组件生命周期等用等问题。

摘要图（来自[深入理解Android插件化技术](http://www.bieryun.com/1348.html)）

![](https://i.imgur.com/vEhNmk7.jpg)

插件可能是apk也可以能so格式，都不会生成R.id. 这个问题有好几种解决方案

1. 重写Context的getAsset、getResource之类的方法，偷换概念，让插件读取插件里的资源，但缺点就是宿主和插件的资源id会冲突，需要重写AAPT
2. 重写AMS中保存的插件列表，从而让宿主和插件分别去加载各自的资源而不会冲突
3. 打包后，执行一个脚本，修改生成包中资源id

[Android插件化：从入门到放弃](http://www.infoq.com/cn/articles/android-plug-ins-from-entry-to-give-up)

[Android插件化的过去，现在，未来](https://www.kymjs.com/code/2016/05/04/01/)

# 专有名词

- 插件化 – apk 分为宿主和插件部分，插件在需要的时候才加载进来
- 热修复 – 通常只包括一个类的方法，体量很小，和正常的插件不是一个数量级的，所以只称为热修复补丁包。一般用于修复bug
- Instant Run – 2016 Google 的 Android Studio 推出了Instant Run热更新功能 同时提出了3个名词
    - “热部署” – 方法内的简单修改，无需重启app和Activity
    - “暖部署” – app无需重启，但是activity需要重启，比如资源的修改
    - “冷部署” – app需要重启，比如继承关系的改变或方法的签名变化等

# 插件化的原理

插件化的目的就是要减小宿主程序apk包的大小同时降低宿主程序的更新频率并做到自由装载模块。包括插件dex如何加载到内存和资源加载问题

有些是使用类加载方案，有些是底层替换方案

可以看这篇[深入理解Android插件化技术](http://www.bieryun.com/1348.html)

还有这篇提到的[技术流派](http://www.infoq.com/cn/articles/android-plug-ins-from-entry-to-give-up)

# 热修复

热修复技术在近年来飞速发展，尤其是在Google InstantRun方案推出之后，各种热修复技术竞相涌现

热修复通常修改很小，是轻量级的插件化，一般用于修复bug

- Google InstantRun
    - InstantRun可以看郭霖的[你真的了解Instant Run吗](http://blog.csdn.net/guolin_blog/article/details/51271369)
    - InstantRun的实现原理：通过给原类添加$change字段为补丁类：和nuwa类似，一个插件一个库。[插件gradle plugin 2.0.0-alpha1和库instant-run.jar](http://jiajixin.cn/2015/11/25/instant-run/)
- 热修复技术
    - [【腾讯WeTest干货分享】全面了解Android热修复技术](https://zhuanlan.zhihu.com/p/29957151)

# 三方框架

目前国内支付宝，淘宝，微信，QQ空间，饿了么，美丽说蘑菇街，美团大众点评等都推出了自己的[热修复方案](https://zhuanlan.zhihu.com/p/25863920)

对比图片（来自网络）

![](https://i.imgur.com/xxZhqJX.png)

[各大热补丁方案分析和比较](http://blog.zhaiyifan.cn/2015/11/20/HotPatchCompare/)

[微信Android热补丁实践演进之路](https://github.com/WeMobileDev/article/blob/master/%E5%BE%AE%E4%BF%A1Android%E7%83%AD%E8%A1%A5%E4%B8%81%E5%AE%9E%E8%B7%B5%E6%BC%94%E8%BF%9B%E4%B9%8B%E8%B7%AF.md#rd)

[Android热修复原理（一）热修复框架对比和代码修复](https://blog.csdn.net/itachi85/article/details/79522200)

PS: [AOP](https://baike.baidu.com/item/AOP/1332219?fr=aladdin) - Aspect Oriented Programming (面向切面编程)

# Reference

经典：

[深入理解Android插件化技术](http://www.bieryun.com/1348.html)

[Android博客周刊专题之＃插件化开发＃](http://www.androidblog.cn/index.php/Index/detail/id/16#)

其他：

[Android插件化技术——原理篇](http://mp.weixin.qq.com/s/Uwr6Rimc7Gpnq4wMFZSAag)

[【腾讯WeTest干货分享】全面了解Android热修复技术](https://zhuanlan.zhihu.com/p/29957151)

[Android 热修复专题](https://zhuanlan.zhihu.com/p/25863920)

[各大热补丁方案分析和比较](http://blog.zhaiyifan.cn/2015/11/20/HotPatchCompare/)

[Android插件化：从入门到放弃](http://www.infoq.com/cn/articles/android-plug-ins-from-entry-to-give-up)

[微信Android热补丁实践演进之路](https://github.com/WeMobileDev/article/blob/master/%E5%BE%AE%E4%BF%A1Android%E7%83%AD%E8%A1%A5%E4%B8%81%E5%AE%9E%E8%B7%B5%E6%BC%94%E8%BF%9B%E4%B9%8B%E8%B7%AF.md#rd)

[Android 动态链接库加载原理及 HotFix 方案介绍](http://mp.weixin.qq.com/s/wvt3NABA-NnQxpbcxhAGiA)

[【新技能get】让App像Web一样发布新版本](http://mp.weixin.qq.com/s/jcBUs4TSRBLkl-Jm0Ye31g)
