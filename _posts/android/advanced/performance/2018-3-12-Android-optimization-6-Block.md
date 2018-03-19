---
layout: post
title: 性能优化（六）卡顿，应用流畅度监测
date: 2018-3-5
excerpt: "卡顿，应用流畅度监测"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

在Android项目中应用的流畅度是UX的重要标准之一，负责的App界面，UI线程的耗时等都有可能会导致卡顿。

严重的卡顿还有可能造成ANR，这个在[性能优化方法（一）方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)中有提到

前面的文章有讲过Android的渲染机制：

系统每隔16ms发出VSYNC信号，触发对UI进行渲染。

如果某个操作花费时间>16ms，系统在得到VSYNC信号的时候就无法进行正常渲染。就发生了丢帧现象。那么用户在32ms内看到的会是同一帧画面。

就出现了卡顿不流畅。

大多数用户感知到的卡顿等性能问题的最主要根源都是因为渲染性能，大量的IO操作或是大量的计算操作占用CPU。

渲染优化，比如没有必要的layouts、invalidations、Overdraw这些在前面的篇幅中已经提到。这里我主要来看如何监测线程中的卡顿

# 卡顿监测 

一般主线程过多的UI绘制、大量的IO操作或是大量的计算操作占用CPU，导致App界面卡顿。我们需要在发生卡顿的时候，捕捉到主线程的堆栈信息和系统的资源使用信息来分析卡顿。

目前已有的两种主流有效的app监控方式

1. 利用UI线程的Looper打印的日志匹配

    原理是利用UI线程有Looper的dispatchMessage前后两个log的输出时间差就能检测到部分UI线程是否有耗时的操作。
    
    即logging，的两个输出：”>>>>> Dispatching to “和”<<<<< Finished to “两次log的时间差值，来计算dispatchMessage的执行时间，从而设置阈值判断是否发生了卡顿。

        ......
        for (;;) {
                Message msg = queue.next(); 
                // might block
                if (msg == null) {
                // No message indicates that the message queue is quitting.
                    return;
                }
                // This must be in a local variable, in case a UI event sets the logger
                Printer logging = me.mLogging;        
                if (logging != null) {
                    // 分发前
                    logging.println(">>>>> Dispatching to " + msg.target + " " +            
                    msg.callback + ": " + msg.what);
                }
    
                msg.target.dispatchMessage(msg);        
                if (logging != null) {
                    // 分发后
                    logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
                }        
                ......
         }
    }
    
    可以通过Looper的setMessageLogging方法设置logging (实例摘抄自网上）
    
    如果UI线程阻塞超过1000ms，就会在子线程中执行mLogRunnable，打印出UI线程当前的堆栈信息，如果处理消息没有超过1000ms，则会实时的remove掉这个mLogRunnable任务。
    
        public class BlockDetectByPrinter {    
            public static void start() {
                    Looper.getMainLooper().setMessageLogging(new Printer() { 
                       private static final String START = ">>>>> Dispatching"; 
                       private static final String END = "<<<<< Finished";
                        @Override
                        public void println(String x) {
                          if (x.startsWith(START)) {
                                LogMonitor.getInstance().startMonitor();
                            }  
                          if (x.startsWith(END)) {
                                LogMonitor.getInstance().removeMonitor();
                            }
                        }
                    });
                }
        }
        
        public class LogMonitor {    
            private static LogMonitor sInstance = new LogMonitor();    
            private HandlerThread mLogThread = new HandlerThread("log");    
            private Handler mIoHandler;    
            private static final long TIME_BLOCK = 1000L;    
            private LogMonitor() {
                    mLogThread.start();
                    mIoHandler = new Handler(mLogThread.getLooper());
                } 
               
            private static Runnable mLogRunnable = new Runnable() {        
                    @Override
                    public void run() {
                        StringBuilder sb = new StringBuilder();
                        StackTraceElement[] stackTrace = Looper.getMainLooper().getThread().getStackTrace();            
                        for (StackTraceElement s : stackTrace) {
                            sb.append(s.toString() + "\n");
                        }
                        Log.e("TAG", sb.toString());
                    }
                };
                
                public static LogMonitor getInstance() {        
                    return sInstance;
                }    
                public boolean isMonitor() {        
                    return mIoHandler.hasCallbacks(mLogRunnable);
                }    
                public void startMonitor() {
                    mIoHandler.postDelayed(mLogRunnable, TIME_BLOCK);
                }    
                public void removeMonitor() {
                    mIoHandler.removeCallbacks(mLogRunnable);
                }
            }



2. 使用[Choreographer.FrameCallback](https://developer.android.com/reference/android/view/Choreographer.FrameCallback.html) （Android 4.1 API 16以上支持）


    Choreographer类的FrameCallback是系统每隔16ms发出VSYNC信号，来通知界面进行重绘、渲染的回调。利用两次回调的时间差十分超出16ms来判断卡顿
 
    当每一帧被渲染时会触发回调FrameCallback， FrameCallback回调void doFrame (long frameTimeNanos)函数。检查两次doFrame的间隔，如果大于16.6ms说明发生了卡顿
    
        public class BlockDetectByChoreographer {
            public static void start() {
                Choreographer.getInstance().postFrameCallback(new Choreographer.FrameCallback() { 
                        long lastFrameTimeNanos = 0; 
                        long currentFrameTimeNanos = 0;
        
                        @Override
                        public void doFrame(long frameTimeNanos) { 
                            if(lastFrameTimeNanos == 0){
                                lastFrameTimeNanos == frameTimeNanos;
                            }
                            currentFrameTimeNanos = frameTimeNanos;
                            long diffMs = TimeUnit.MILLISECONDS.convert(currentFrameTimeNanos-lastFrameTimeNanos, TimeUnit.NANOSECONDS);
                            if (diffMs > 16.6f) {            
                               long droppedCount = (int)diffMs / 16.6;
                            }
                            if (LogMonitor.getInstance().isMonitor()) {
                                LogMonitor.getInstance().removeMonitor();                    
                            } 
                            LogMonitor.getInstance().startMonitor();
                            Choreographer.getInstance().postFrameCallback(this);
                        }
                });
            }
         }

# 优秀的轮子

[BlockCanary](https://github.com/markzhai/AndroidPerformanceMonitor)：使用的是第一种方式，两笔logging的时间差

# Reference

[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[广研Android卡顿监控系统](http://mp.weixin.qq.com/s/MthGj4AwFPL2JrZ0x1i4fw)







