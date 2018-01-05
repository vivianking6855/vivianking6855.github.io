---
layout: post
title: Contact 优化 - 开篇
date: 2017-12-26
excerpt: "Contact 优化 - 开篇"
categories: Projects
tags: [Projects]
comments: true
---


* content
{:toc}




# 简介

关于性能优化的问题，主要关注的有：

- 内存
- CPU
- 耗电
- 卡顿
- 渲染
- 进程存活率等

性能优化需要注意：

1. 不要过早的做性能优化，app先求能用再求好用。在需求都还没完成的时候，花大量时间在优化上是本末倒置的
2. 优化要用实际数据说话，建议借助测试工具进行检测。检测工具参看[这里](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)

总之，要合理优化，数据量化。

# Contact优化

我们针对联系人做的优化大致如下：

1. size优化

    移除unused resources，降低app的size

2. 布局，绘制，响应速度等性能优化

    - 主页面，联络人详情优化: 主要是布局优化，绘制优化，响应速度优化
    - call log滑动: 主要是布局优化，绘制优化，响应速度优化
    - smart search: 内存和响应速度优化

3. [重构](http://vivianking6855.github.io/2017/03/30/Android-Design-Refactoring/)，在现有的框架上适当重构。

4. 其他
    - zipAlignEnabled: 对应用程序中的资源作对齐操作 
    
        使得在运行时Android与应用程序间的交互更加有效率。这种方式能够让应用程序和整个系统运行得更快
    
            buildTypes {
                release {
                    shrinkResources true
                    minifyEnabled true
                    zipAlignEnabled true
                    proguardFiles 'proguard.flags'
                }
            }

# Reference

[Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

[性能优化（一）：方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)

[性能优化（二）内存管理 & Memory Leak & OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)

[性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)

[性能优化（四）Google典范之Render实践](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

[性能优化 - 工具](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)