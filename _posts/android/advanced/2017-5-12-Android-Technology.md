---
layout: post
title: Android应用开发需精通20个技能点
date: 2017-5-12
excerpt: "Android应用开发需精通20个技能点"
categories: Android
tags: [Android 进阶]
comments: true
---

# 前言

做Android应用开发，不需要花太多精力去看Android系统源码，要先确保下面的20点所涉及的技术都掌握了。

## 1. Activity相关

App应用开发以Activity使用最多，涉及LaunchMode, onSaveInstance-State, 生命周期等技术

## 2. Fragment相关

推荐一本书[《Creating Dynamic UI with Android Fragments》](https://book.douban.com/subject/25796846/)

## 3. 序列化技术

有Parcelable和Serializable两种。

前者是基于Service的，后者是基于Bundle的，二者实现原理不同，但效果差不多

## 4. ImageLoader的原理和使用。

类似的还可以学习Facebook新进开源的Fresco，非常优秀的图片处理库

## 5. fastJSON和GSON的使用。

做App需要会用实体自动匹配JSON数据

## 6. 多线程

Handler、Looper、ExecutorService、RxAndroid、RxJava等

## 7. Adapter和ListView

这两个在一起经常crash，特别是分页的处理，尤其需要仔细研究深刻体会

## 8. 用户Cookie设计。

需要把登录机制彻底搞清楚，包括在HttpRequest头中夹带Cookie来进行用户身份验证的技术

## 9. 网络请求封装

使用AsyncTask的网络底层封装，使用Handler + Runnable的网络底层封装。

也可以使用成熟的三方库，例如Retrofit、OKhttp等

## 10. Android与H5的交互

包括Android和H5方法的互相调用

## 11. 代码混淆

ProGuard，keep语法等

有些project可能只压缩不混淆：-dontobfuscate

## 12. Android打包机制

涉及Android多项目依赖的打包技术。 Ant、Gradle或者Maven, 掌握其中一种打包机制即可。

## 13. 线上Crash分析并修复

要具备通过分析Crash信息修复线上Crash的能力

## 14. 内存泄漏

包括内存优化、内存泄漏的场景、MAT工具的使用

## 15. 调试工具

包括DDMS、Eclipse或AS的调试功能

## 16. Monkey机制

如何对一款App进行Monkey测试。

## 17. 单元测试

JUnit，对复杂的算法写过单元测试以确保其没有问题

## 18. GIT的功能

包括Stage、Rebase、Revert、Stash、Cherry Pick和Stub Module等概念。 建议使用smartgit工具简化操作。

## 19. 插件化编程

DexClassLoader的概念，了解插件化编程的思想和做法

## 20. 设计模式

对常见的设计模式如工厂、生成器、适配器、代理、策略模式耳熟能详。

## 面试考察

RD

- 1） Activity的生命周期
- 2） Activity的4中启动方式及使用场合
- 3） 做过的项目中Activity是否有基类。如果有封装了哪些共用的逻辑
- 4） 事件的各种使用方式及优缺点
- 5） 与H5页面的相互调用
- 6） UI线程的阻塞与解决方案（Runnable和Handler）
- 7） 采用什么姿势调用MobieAPI并接卸返回的数据
- 8） 怎样做列表的分页和刷新
- 9） 登录的实现： 包括从哪儿来，到哪儿去的页面跳转机制，记住密码的逻辑设计。
- 10）性能调优
    - Layout调优
    - Activity中如何使用CONST常量
    - 时间换空间策略
    - ViewHolder
    - 图集的优化策略
    - 数据缓存和图片缓存
- 11）全局变量过多怎么办
- 12）UT
- 13）自动打包。Ant、Maven或Gradle任意一种都可以。

TeamLeader

- 检查内存泄漏
- 优化内存
- 多线程
- 自动打包
- 框架设计
- 版本管理

# 推荐书籍

- 《Android研发录》
- 《疯狂的Android讲义》： 适合于入门教材，入门也可以梳理下知识
- 《Creating Dynamic UI with Android Fragments》： 专门讲Fragment的书，非常详细 
- 《Android应用测试与调试实践》： 书中很多章节涉及依赖逐日、内存分析、打包部署等技术
- 《Java与模式》： 古董级的书。介绍设计模式。
- 《Git权威指南》： 详细介绍Git的书。




