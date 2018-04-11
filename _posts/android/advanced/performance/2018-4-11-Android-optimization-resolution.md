---
layout: post
title: 移动应用优化解决方案
date: 2018-4-11
excerpt: "移动应用优化解决方案"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# 前言

移动应用优化是产品中非常重要，也是比不可少的一环。

Android app优化主要包含size优化，性能优化，重构等。

1. size优化，app减负是必不可少的一环。
2. 性能优化，提升UX体验的重要途径（当然Spec的定义也会直接影响UX）。性能优化主要关注
3. 重构：业务复杂度提示，需要重构提高易测性和扩展性，降低耦合

其中性能优化又是特别重要的一环

性能主要关注：

1. lanch time
2. 渲染
3. 内存
4. 耗电和网络
5. 卡顿
6. 数据库优化
7. 进程存活率

下面是针对Android端应用来阐述的优化解决方案。

# 方案

方案的核心：客户端搭配服务器端实现细致的监控，一起实现闭环的解决方案：监控，大数据统计和分析，回馈研发，解决，发布再到监控。

![](https://i.imgur.com/oYPZtTe.jpg)

最终的目标做到合理优化，数据的量化。 更重要是如何建立合理的框架避免发生问题，或者是能及时的发现问题。

文章的最后还说明了发布和测试需要注意的一些事项

# 监控

服务器端应该监控的模块有：

1. app基本统计信息（app版本，用户活跃度，地区，设备信息，页面活跃度，安装量，留存，功能使用等）
2. 异常监控(包含Crash,ANR）
3. 内存监控（例如有版本的平均内存，已经版本增幅的数据可以清晰看到量化的效果；包含OOM监控）
4. 耗电监控
5. 缓存监控


现在主流监控平台基本都实现了1 app统计信息和2 异常监控（例如友盟，GA，Bugly，Testin），这里不做赘述。

有些平台有做内存，电量，流量监控。例如：

- 网易的Emmagee： CPU、内存、流量、电量等
- 腾讯的GT： CPU、内存、流量、电量、帧率/流畅度等等
- 腾讯的APT：CPU，内存检测，内存构成分析
- 科大讯飞的iTest： 记录特定应用的性能消耗情况，包括cpu、内存、流量、电量等信息

不过缓存监控，截止现在暂时没发现可以接入的平台。腾讯Bugly有针对自己的手Q做这些监控，不过没有公开（记录时间2018/4/11)

# 1. size优化

参看[Contact size优化篇](http://vivianking6855.github.io/2017/12/27/Contact-Optimization-2/)

备注：这里没有设计so的优化，待以后有实战了再更新

# 2. 性能优化

## 2.1 lanch time

参看[性能优化（八）启动时间优化](http://vivianking6855.github.io/2018/04/11/Android-optimization-8-Launch-Time/)

## 2.2 渲染

渲染操作通常依赖于两个核心组件：CPU与GPU。CPU和GPU的工作，潜在的问题，检测的工具和解决方案图：
 
![](http://i.imgur.com/SiZVlJ9.png)

参看

[性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)

[性能优化（四）Google典范之Render实践](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

[Contact 优化 - detail页面优化](http://vivianking6855.github.io/2018/01/04/Contact-Optimization-3/)

## 2.3 内存

## 明显的内存问题

比较明显的内存问题，例如

1. 明显的MemoryLeak
2. 必发的内存不合理分配
3. 内存抖动
4. GC引起的性能问题（例如循环单元里面执行了创建对象操作，短时间内发生大量的内存上涨和降低事件）

我们可以用一些工具和技术监测分析以及代码，逻辑优化来解决

参看下面三篇

[性能优化：要点](http://vivianking6855.github.io/2018/01/24/Android-optimization-critical/)

[性能优化（五）内存优化](http://vivianking6855.github.io/2018/03/05/Android-optimization-5-Memory/)

[Contact 优化 - detail页面优化](http://vivianking6855.github.io/2018/01/04/Contact-Optimization-3/)

## 更深入的内存分析

比如偶发或长期操作才可能出现的不明显内存问题，就需要更深入的内存分析。这个需要搭配服务器端的细致监控app平均内存，平均OOM率，以及版本间增幅等相关的深入分析，就需要搭配服务器端。

需要结合内存监控和缓存监控来分析和解决这部分内存和OOM

## 2.4. 耗电和网路

参看[性能优化（七）电量和网络优化](http://vivianking6855.github.io/2018/04/11/Android-optimization-7-Power/)

## 2.5 卡顿

参看[性能优化（六）卡顿监测](http://vivianking6855.github.io/2018/03/05/Android-optimization-6-Block/)

## 2.6 数据库优化

笔者的项目暂不需要，待以后实作会补充上来。

一些建议可以参看：[性能优化之数据库优化](http://www.trinea.cn/android/database-performance/)

主要是索引优化，使用事务，sqlite优化，异步单线程等

## 2.7 进程存活率

参看这里[Android进程保活招式大全](https://blog.csdn.net/tencent_bugly/article/details/52192423)

# 如何避免性能问题

在优化的同时，要知道更重要是如何建立合理的框架避免发生问题，或者是能及时的发现问题

- 合理的框架，避免资源没有释放问题。例如我们项目中使用的AppUniform, MVP的P会在destroy时自动释放
- 例如独立电量管理框架，自动释放内存

另外程序的Code Style要尽量遵循[性能优化：要点](http://vivianking6855.github.io/2018/01/24/Android-optimization-critical/)

# 压力测试

- monkey > 18小时，生成报表：crash，ANR，内存图标等
- 建议使用Espresso，UiAutomator，AndroidJunitRunner，JUnit4等创建测试用例

# 方案发布

正式方案发布需要特别注意下面的几项：

1. 严密的质量测试和压力测试
2. 尽量阶梯式发布 0.02， 0.5
3. 实时监测发布后的状态

# 小结

1.不要过早的做性能优化，app先求能用再求好用。在需求都还没完成的时候，花大量时间在优化上是本末倒置的
2.优化要用实际数据说话，建议借助测试工具进行检测。

总之，要合理优化，数据量化。而且搭配服务器做大数据统计和分析是非常必要的。


# Reference

[性能优化 - 目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)









