---
layout: post
title: Android ANR
date: 2018-1-30
excerpt: "Android ANR"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# ANR出现的原因

1. KeyDispatchTimeout(5 seconds) --主要类型按键或触摸事件在特定时间内无响应
2. BroadcastTimeout(10 seconds) --BroadcastReceiver在特定时间内无法处理完成
3. ServiceTimeout(20 seconds) --小概率类型 Service在特定的时间内无法处理完成

容易出问题几个点：

1. Activity 生命周期
2. View post 的runnable方法 、 handler（MainLooper） 的 handleMessage()
3. Asycktask 的 onPreExecute(), onPostExecute() , onProgressUpdate()方法
4. Broadcast 的onReceive()
5. Service
6. 线程死锁

几个容易发生ANR的场景：

- 最常见的错误，UI线程等待其它线程释放某个锁，导致UI线程无法处理用户输入；
- 游戏中每帧动画都进行了比较耗时的大量计算，导致CPU忙不过来；
- Web应用中，网络状态不稳定，而界面在等待网络数据；
- UI线程中进行了一些磁盘IO（包括数据库、SD卡等等）的操作，在个别设备上因为硬件损坏等原因阻塞住了；
- 手机被其他App占用着CPU，自己获取不到足够的CPU

# ANR 解析

anr解析通常是logcat文件搭配trace文件(系统自动生成)/console_error.log文件(我们使用的monkey工具命名)

系统的trace文件通常在：data/anr/trace.txt

通常先查看main thread的状况，然后看是否有waiting on, locked等关键字

## 案例一 "main" Blocked

查看trace文件：main线程 Blocked

    Cmd line: com.asus.cncontacts
    
    ......

    "main" prio=5 tid=1 Blocked
      | group="main" sCount=1 dsCount=0 obj=0x74c5baf8 self=0x7fa9c96a00
      | sysTid=3519 nice=-10 cgrp=default sched=0/0 handle=0x7fadc80a98
      | state=S schedstat=( 4583926025 403425081 5667 ) utm=356 stm=102 core=5 HZ=100
      | stack=0x7fd1250000-0x7fd1252000 stackSize=8MB
      | held mutexes=
      at com.android.contacts.smartdial.ContactsListSearcher.resetAll(ContactsListSearcher.java:36)
      - waiting to lock <0x0a4f1e7b> (a com.android.contacts.smartdial.ContactsListSearcher) held by thread 43 *************

可以看到*标注的地方显示，在等待thread 43锁住了。接着搜索tid=43

    "Thread-68" prio=5 tid=43 Waiting
      | group="main" sCount=1 dsCount=0 obj=0x13a40ec0 self=0x7f89588000
      | sysTid=5286 nice=0 cgrp=default sched=0/0 handle=0x7f874ee450
      | state=S schedstat=( 770834 332500 4 ) utm=0 stm=0 core=7 HZ=100
      | stack=0x7f873ec000-0x7f873ee000 stackSize=1037KB
      | held mutexes=
      at java.lang.Object.wait!(Native method)
      - waiting on <0x0191dc62> (a java.lang.Object) ************************
      at java.lang.Thread.parkFor$(Thread.java:2127)
      - locked <0x0191dc62> (a java.lang.Object) 
      at sun.misc.Unsafe.park(Unsafe.java:325)
      at java.util.concurrent.locks.LockSupport.park(LockSupport.java:161)
      at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:840)
      at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:994)
      at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1303)
      at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:203)
      at com.android.contacts.smartdial.UnicodeToPinyin.map(UnicodeToPinyin.java:31) ===============
      at com.android.contacts.smartdial.DialpadMapper.map(DialpadMapper.java:106)
      at com.android.contacts.smartdial.DialpadMapper.map(DialpadMapper.java:142)
      at com.android.contacts.smartdial.ContactsListSearcher$ContactWordsInfo.<init>(ContactsListSearcher.java:340)
      at com.android.contacts.smartdial.ContactsListSearcher.addContact(ContactsListSearcher.java:158)
      - locked <0x0a4f1e7b> (a com.android.contacts.smartdial.ContactsListSearcher)

可以看到*标注的地方显示，在等待<0x0191dc62>，这个被=标注的UnicodeToPinyin.java:31锁住了。

root cause: 不是所有情景锁都有释放。context 为null是没有释放锁

## 案例二 Waiting because no window ......

需要查看trace文件

    // NOT RESPONDING: com.asus.cncontacts (pid 3166)
    ANR in com.asus.cncontacts (com.asus.cncontacts/com.android.contacts.activities.SortActivity)
    PID: 3166
    Reason: Input dispatching timed out (Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)
    Load: 7.19 / 6.9 / 6.44
    
    ......
    
    "main" prio=5 tid=1 Native
      | group="main" sCount=1 dsCount=0 obj=0x74694af8 self=0x7f85c96a00
      | sysTid=3166 nice=-10 cgrp=default sched=0/0 handle=0x7f89bc6a98
      | state=S schedstat=( 3328701532182 149508422359 2999128 ) utm=273800 stm=59070 core=7 HZ=100
      | stack=0x7fd8433000-0x7fd8435000 stackSize=8MB
      | held mutexes=
      kernel: __switch_to+0x7c/0x88
      kernel: SyS_epoll_wait+0x2e8/0x398
      kernel: SyS_epoll_pwait+0xc4/0x150
      kernel: el0_svc_naked+0x24/0x28
      native: #00 pc 000000000006a84c  /system/lib64/libc.so (__epoll_pwait+8)
      native: #01 pc 000000000001e534  /system/lib64/libc.so (epoll_pwait+64)
      native: #02 pc 0000000000017f98  /system/lib64/libutils.so (_ZN7android6Looper9pollInnerEi+156)
      native: #03 pc 0000000000017e4c  /system/lib64/libutils.so (_ZN7android6Looper8pollOnceEiPiS1_PPv+60)
      native: #04 pc 00000000000f2674  /system/lib64/libandroid_runtime.so (_ZN7android18NativeMessageQueue8pollOnceEP7_JNIEnvP8_jobjecti+48)
      native: #05 pc 00000000008ce1d0  /system/framework/arm64/boot-framework.oat (Java_android_os_MessageQueue_nativePollOnce__JI+140)
      at android.os.MessageQueue.nativePollOnce(Native method)
      at android.os.MessageQueue.next(MessageQueue.java:323) **************
      at android.os.Looper.loop(Looper.java:139)
      at android.app.ActivityThread.main(ActivityThread.java:6139)
      at java.lang.reflect.Method.invoke!(Native method)
      at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:886)
      at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:776)

从*标注的位置来看卡在取messeage的地方，结合Reason来看可能是app window很久还没创建成功，导致系统的发的Message一直没有被取走

root cause：Activity创建时可能有非常耗时的操作。查看onCreate中有访问DB的动作。
    
## 案例三 cpu超负载 （摘录自Android ANR 分析学习总结）

需要trace文件搭配logcat

    04-0113:12:15.872 E/ActivityManager( 220): CPUusage from 4361ms to 699ms ago ----CPU在ANR发生前的使用情况

    04-0113:12:15.872 E/ActivityManager( 220): 5.5%21404/com.android.email: 1.3% user + 4.1% kernel / faults:
    10 minor
    04-0113:12:15.872 E/ActivityManager( 220): 4.3%220/system_server: 2.7% user + 1.5% kernel / faults: 11
    minor 2 major
    04-0113:12:15.872 E/ActivityManager( 220): 0.9%52/spi_qsd.0: 0% user + 0.9% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 0.5%65/irq/170-cyttsp-: 0% user + 0.5% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 0.5%296/com.android.systemui: 0.5% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 100%TOTAL: 4.8% user + 7.6% kernel + 87% iowait ********************
    04-0113:12:15.872 E/ActivityManager( 220): CPUusage from 3697ms to 4223ms later:-- ANR后CPU的使用量
    04-0113:12:15.872 E/ActivityManager( 220): 25%21404/com.android.email: 25% user + 0% kernel / faults: 191 minor
    04-0113:12:15.872 E/ActivityManager( 220): 16% 21603/__eas(par.hakan: 16% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 7.2% 21406/GC: 7.2% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 1.8% 21409/Compiler: 1.8% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 5.5%220/system_server: 0% user + 5.5% kernel / faults: 1 minor
    04-0113:12:15.872 E/ActivityManager( 220): 5.5% 263/InputDispatcher: 0% user + 5.5% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 32%TOTAL: 28% user + 3.7% kernel

看中间注释，还有*标注的地方有： 100%TOTAL: 4.8% user + 7.6% kernel + 87% iowait

说明cpu使用率100%。我们再搭配trace文件，*标注代表handle在等待取消息

    DALVIK THREADS:
      (mutexes: tll=0tsl=0 tscl=0 ghl=0 hwl=0 hwll=0)
      "main" prio=5 tid=1NATIVE
      | group="main" sCount=1 dsCount=0obj=0x2aad2248 self=0xcf70
      | sysTid=21404 nice=0 sched=0/0cgrp=[fopen-error:2] 
      handle=1876218976 **********
      atandroid.os.MessageQueue.nativePollOnce(Native Method)
      atandroid.os.MessageQueue.next(MessageQueue.java:119) *******
      atandroid.os.Looper.loop(Looper.java:110)
      at android.app.ActivityThread.main(ActivityThread.java:3688)
      at java.lang.reflect.Method.invokeNative(Native Method)
      atjava.lang.reflect.Method.invoke(Method.java:507)
    
      atcom.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:866)
      at 
      com.android.internal.os.ZygoteInit.main(ZygoteInit.java:624)
      at dalvik.system.NativeStart.main(Native Method)

# Reference

> [【腾讯bugly干货分享】精神哥手把手教你如何智斗ANR](http://blog.csdn.net/tencent_bugly/article/details/46650675)
 
> [Android ANR 分析学习总结](http://blog.csdn.net/nothingl3/article/details/52800182)

> [Android ANR问题总结（一）](http://blog.csdn.net/jiangguohu1/article/details/52636470)
