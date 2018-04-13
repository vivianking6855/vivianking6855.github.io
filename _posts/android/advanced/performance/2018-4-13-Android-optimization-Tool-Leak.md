---
layout: post
title: 性能优化 - Memory Leak 分析
date: 2018-4-13
excerpt: "性能优化 - Memory Leak 分析"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# 内存泄漏

Memory Leak 内存泄露的说明前面一篇有详细解释[性能优化（二）内存管理 & Memory Leak & OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)

Memory Leak简单的说就是无用的对象占据着内存空间，没有合理释放，使得实际可使用内存变小，导致内存泄漏。

本文主要是展示分别用AS和MAT工具如何分析内存泄漏。

# 准备泄漏测试代码

核心Code

    private void testMemoryLeak() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(60 * 60 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }, "VivianLeakThread").start();
    }

Github Sample [地址](https://github.com/vivianking6855/android-advanced/tree/master/CodeDebug)

app启动，退出，如此循环3,4次。

Okay，环境准备好了，现在我们开始实战吧！

# Android Studio分析

gc两次后，dump heap

   ![](https://i.imgur.com/YiqUfYq.jpg)
    
按照报名排序，或者直接搜索MainActivity。看到红色部分，MainActivity被Thread持有导致很多个MainActivity没有被释放。
    
# MAT分析

1. 从AS导出hprof



2. hprof-conv工具转换成MAT可识别的格式

    hprof-conv heap-original.hprof heap-converted.hprof

3. MAT打开文件（文件最好放在一个独立的文件夹里面，因为MAT分析的时候会产生很多的临时文件）

    打开Histogram，搜索MainActivity

    ![](https://i.imgur.com/JRatiX1.jpg)

    选择排除所有的幽灵，soft,weak reference
    
    ![](https://i.imgur.com/ln7pGsP.jpg)

    可以看到因为thead持有MainActivity的强引用，MainActivity无法释放，造成了内存泄露。
    
    ![](https://i.imgur.com/5trBYpE.jpg)


# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)