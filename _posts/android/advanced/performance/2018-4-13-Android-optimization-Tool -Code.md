---
layout: post
title: 性能优化 - 代码调试工具
date: 2018-4-13
excerpt: "性能优化 - 代码调试工具"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}


# 代码调试工具

虽然AS有提供很好的GUI可以分析CPU, Memory等，但是时间点很难准确的捕获。

这是就需要搭配代码调试一起分析。 这几介绍几种在代码中如何监测性能的方案。

# 启动耗时间

启动耗时间监测的三种方法：

1. Android 4.4之后可以在logcat中查看TAG “ActivityManager” 信息可以看到Activity启动时间

    critical tag： "ActivityManager: Displayed" ; 设置输出level：Info

        04-13 10:32:33.380 1584-1742/system_process I/ActivityManager: Displayed com.vv.learn.debugdemo/.MainActivity: +1s760ms [aosp]

2. 参照从OnCreate到onWindowFocusChanged的时间差

        @Override
        public void onWindowFocusChanged(boolean hasFocus) {
            super.onWindowFocusChanged(hasFocus);
    
            if (hasFocus) {
                currentTimeNanos = System.nanoTime();
                long diffMs = TimeUnit.MILLISECONDS.convert(currentTimeNanos - lastTimeNanos, TimeUnit.NANOSECONDS);
                Log.d(TAG, "onWindowFocusChanged .MainActivity: +" + diffMs + "ms");
            }
        }
        
        输出：

        04-13 10:56:15.448 21100-21100/com.vv.learn.debugdemo D/vv: .MainActivity: +100ms

    这个时间是比系统的统计的要短

3. 页面完全加载可以参照从OnCreate到到IdleHandler的时间差


        // IdleHandler to collect diffMs of activity launch nano time
        Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
            @Override
            public boolean queueIdle() {
                currentTimeNanos = System.nanoTime();
                long diffMs = TimeUnit.MILLISECONDS.convert(currentTimeNanos - lastTimeNanos, TimeUnit.NANOSECONDS);
                Log.d(TAG, "IdleHandler .MainActivity: +" + diffMs + "ms");
                return false;
            }
        });
        
        输出：

        04-13 11:07:16.669 27270-27270/com.vv.learn.debugdemo D/CodeDebug: onCreate MethodTracing: 2ms
        04-13 11:07:16.904 27270-27270/com.vv.learn.debugdemo D/CodeDebug: onWindowFocusChanged .MainActivity: +236ms
        04-13 11:07:16.927 27270-27270/com.vv.learn.debugdemo D/CodeDebug: IdleHandler .MainActivity: +259ms
 
# TraceView - MethodTracing

上面的时间主要来分析整个启动时间，更详细的method耗时，需要用到Debug.startMethodTracing

核心代码

    private final static String TAG = "CodeDebugUtils";
    private final static String TRACING_FILE = "method.trace";

    /**
     * Start tracing long.
     *
     * @param context the context
     * @return the long nanos time of start time
     */
    public static void startTracing(final Context context) {
        final String path = PathUtils.getDiskCacheDir(context, TRACING_FILE).getPath();
        Debug.startMethodTracing(path);
        if (BuildConfig.DEBUG) Log.d(TAG, "MethodTracing path: " + path);
    }

    /**
     * Stop tracing without collect method tracing nano time
     */
    public static void stopTracing() {
        Debug.stopMethodTracing();
    }


PS：我在测试时发现方法只能run在UI线程，子线程无法生成文件。原因待有空研究源码再更新（如果有大牛知晓，还请不吝赐教！）

而且因为TraceView有IO操作，存在严重的运行时开销。不适合放在正式release 专案中。建议debug时使用。

# Systrace

Systrace是分析Android性能问题的神器，可以跟踪系统的I/O操作、内核工作队列、CPU负载以及Android各个子系统的运行状况等

Systrace是Android4.1中新增的性能数据采样和分析工具，适用于4.1及以上的系统。

## 原理

在系统的一些关键链路（比如System Service，虚拟机，Binder驱动）插入一些信息Label，通过Label的开始和结束来确定某个核心过程的执行时间

然后把这些Label信息收集起来得到系统关键路径的运行时间信息，进而得到整个系统的运行性能信息。

Android Framework里面一些重要的模块都插入了Label信息，例如

- Java层的通过android.os.Trace类完成
- native层通过ATrace宏完成
- 用户App中可以添加自定义的Label，这样就组成了一个完成的性能分析系统

## 功能

功能包括：跟踪系统的I/O操作、内核工作队列、CPU负载以及Android各个子系统的运行状况等

它可以帮助开发者收集Android关键子系统：例如Surfaceflinger、WindowManagerService等Framework部分关键模块、服务的运行信息，从而改进性能

在Android平台中，它主要由3部分组成：

1. 内核部分
    - Systrace利用了Linux Kernel中的ftrace功能
    - 使用Systrace必须开启kernel中和ftrace相关的模块
2. 数据采集部分
    - Android定义了一个Trace类，应用程序可利用该类把统计信息输出给ftrace。
    - 还有atrace程序，它可以从ftrace中读取统计信息然后交给数据分析工具来处理
3. 数据分析工具
    - systrace.py: 配置数据采集的方式，例如采集数据的标签、输出文件名等
        - 文件位于sdk\platform-tools\systrace
        - 内部将调用atrace程序
    - 收集ftrace统计数据并生成一个结果网页文件供用户查看

从本质上说，Systrace是对Linux Kernel中ftrace的封装

## 使用

TODO

# 代码dump hrof

Android Profile可以直接监测Memory情况，不过如果需要将hrof上传至服务器，就需要在代码中处理。

可以使用下面的方法（请不要在主线程中做dump）

        FileTask.THREAD_POOL_EXECUTOR.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Debug.dumpHprofData(hpath);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });

# Github Sample [地址](https://github.com/vivianking6855/android-advanced/tree/master/CodeDebug)


# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[手把手教你使用Systrace](https://zhuanlan.zhihu.com/p/27331842)

[Android Systrace的使用]( https://www.aliyun.com/jiaocheng/15246.html)

[Android自动dump hprof文件的功能实现](https://blog.csdn.net/cxq234843654/article/details/51376050)