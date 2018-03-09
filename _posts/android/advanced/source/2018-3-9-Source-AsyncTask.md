---
layout: post
title: 源码解析-AsyncTask
date: 2018-3-9
excerpt: "源码解析-AsyncTask"
categories: 源码解析
tags: [源码解析]
comments: true
---




# 用法

- 继承AsyncTask，重载方法
- 调用execute启动

# 解析总结

核心流程

- 在主线程提交任务到线程池
- 线程池运行doInBackground耗时任务
- 线程池运行完毕后，使用Handler切换到主线程处理结果

![](https://i.imgur.com/5U20G75.jpg)

  
- AsyncTask中CORE_POOL_SIZE大于2小于4:Math.max(2, Math.min(CPU_COUNT - 1, 4));最大POOL_SIZE CPU_COUNT * 2 + 1
- AsyncTask如何并行
    - 调用executeOnExecutor，传入线程池代替sDefaultExecutor，例如executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR)
    - Android5.0是串行执行
    - Android1.6之前和Android3.0之后是串行执行。在这两个版本之间采用的是并行执行

# Reference

[AsyncTask异步任务 源码解读](http://blog.csdn.net/maplejaw_/article/details/51441312)
