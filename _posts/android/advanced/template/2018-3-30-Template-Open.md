---
layout: post
title: Android架构小结
date: 2018-3-30
excerpt: "Android开源架构小结"
categories: Android
tags: [Android 进阶]
comments: true
---


# 简介

为什么需要架构？

当项目非常庞大时，项目的可维护性，可扩展性，易测性显得尤其的重要。

好的架构能够帮助我们更好实现：

•易维护
•易测试
•高内聚
•低耦合

Bob大神的Architecture is About Intent, not Frameworks. 个人理解是：架构应该是面向意图，而不是面向片段。

因此在架构的分层最好依据不同的意图来划分。

另外业务逻辑庞大的项目后期，必须对项目进行模块化的拆分。可以通过组件化，插件化等技术来实现。

这里我们仅针对app的架构讨论，不会涉及深入的组件化和插件化内容。

# 我的AppUniform架构

[AppUniform](https://github.com/vivianking6855/android-advanced/tree/master/AppUniform)是依据之前的项目总结出的Clean Architecture

是我们产品研发时尽可能遵循的原则，我们的期望是任何一个内部环节对外部是解耦的，没有依赖关系。

这也是架构设计的核心：独立性和可测试性；二者是相符想成的。

比如纯Java的可以用JUnit等测试，Android业务相可以用Android Instrumentation等测试

我们的项目基本包含四个层次

![](https://i.imgur.com/5X0VIG0.jpg)

其中数据层，业务无关层分别是独立的library；展示层和业务公有层在app module中

- 展示层，业务展示层采用MVP架构
    - V: activity, fragment, view
    - M: model
    - P: presenter处理数据逻辑，使用data module中的各个api
- 业务公有层 share
    - router 页面跳转器
    - utils 业务公用工具
    - base 业务封装基类
- 数据层 data module
    - api 提供给展示层的数据接口
    - cache 缓存
    - db 数据库
    - entity 数据单元，其中可解析的entity多用于网络请求
    - exception 异常
    - net 网络相关
    - task 各类任务线程池
- 业务无关库 common module
    - 业务逻辑无关的一些公用库
    - 已发到JCenter上的库有：[utilslib](https://bintray.com/vivianwayne1985/maven/utilslib) 和 [appbase](https://bintray.com/vivianwayne1985/maven/appbase)

目录结构如下：

![](https://i.imgur.com/W6LimMp.jpg)


除此之外还有一些建议：

- 尽可能搭配组件化/插件化：注意独立模块的单独调试和继承测试
- 性能和测试
    - 对app核心行为和各个组件交互和生命周期的监控，后期可以搭配bug服务器平台跟trace log一起上传，以便于bug的分析和追踪（LogMan写入文件)
     
# 开源架构

著名的[Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)

查看知乎[的一种更清晰的Android架构](https://zhuanlan.zhihu.com/p/20001838)

# Reference

[一种更清晰的Android架构](https://zhuanlan.zhihu.com/p/20001838)

[Android Architecture](https://github.com/android10/Android-CleanArchitecture)

[Uncle Bob: Architecture is About Intent, not Frameworks](https://www.infoq.com/news/2013/07/architecture_intent_frameworks)