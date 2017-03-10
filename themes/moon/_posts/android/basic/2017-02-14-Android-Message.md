---
layout: post
title: 2017-2-14 Android Messenge
date: 2017-2-14
excerpt: "Android Message"
tags: [Android 基础]
comments: true
---

# Android Message

为什么我们在Android程序中需要多线程？

比如说当用户按下一个Button，你想要做一个非常长时间的操作。如果你不用开启一个额外的线程，那么它看起来会如下代码所示：
    
    ((Button)findViewById(R.id.Button01)).setOnClickListener( 
    newOnClickListener() {
    @Override
    public void onClick(View v) {
    int result = doLongOperation();
    updateUI(result);
    }});

上述代码将会发生什么呢？UI会被冻结阻塞,会出现ANR。这是一个真实的坏的UI体验。

现在我们把长时间操作放在thread中

    ((Button)findViewById(R.id.Button01)).setOnClickListener(
    newOnClickListener() {
    @Override
    public void onClick(View v) {
    (newThread(new Runnable() {
    @Override
    public void run() {
    int result = doLongOperation();
    updateUI(result);
    }})).start();
    }

程序崩溃(crash)，查看源码frameworks/base/core/java/android/view/[ViewRootImpl.java](https://android.googlesource.com/platform/frameworks/base/+/android-7.1.1_r22/core/java/android/view/ViewRootImpl.java)

显然Android并不允许额外的线程来代替主线程修改UI元素. 因为Android UI toolkit, 像其他许多的UI环境一样，它不是线程安全的(hread-safe)。

这时就需要用到Android Message机制。

## Message

介绍Android Message机制前，我们先了解下[Message](https://android.googlesource.com/platform/frameworks/base/+/android-cts-7.1_r2/core/java/android/os/Message.java)。Message是线程之间传递信息的载体，包含了对消息的描述和任意的数据对象。

虽然Message的构造函数是public的，但是最好是使用下面的方法获取Message对象

1.	Message.obtain( )
2.	Handler.obtainMessage( )

因为Message的实现中包含了回收再利用的机制，可以提供效率。


## 消息机制

Android的Message机制主要是指Handler的运行机制已经Handler所附带的MessageQueue和Looper的工作过程。

Android的消息机制三大模块：[Handler,  Looper,  MessageQueue](https://android.googlesource.com/platform/frameworks/base/+/3edcd8c/core/java/android/os/Looper.java)

- MessageQueue消息队列，
- Looper完成消息循环，不断从MessageQueue中获取msg分发

    Looper调用loop后，消息循环才会真正起作用，loop中的一段code
    
        /**
         * Run the message queue in this thread. Be sure to call
         * {@link #quit()} to end the loop.
         */
        public static void loop() {
            final Looper me = myLooper();
            if (me == null) {
                throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
            }
            final MessageQueue queue = me.mQueue;
            // Make sure the identity of this thread is that of the local process,
            // and keep track of what that identity token actually is.
            Binder.clearCallingIdentity();
            final long ident = Binder.clearCallingIdentity();
            for (;;) {
                Message msg = queue.next(); // might block
                if (msg == null) {
                    // No message indicates that the message queue is quitting.
                    return;
                }
                // This must be in a local variable, in case a UI event sets the logger
                Printer logging = me.mLogging;
                if (logging != null) {
                    logging.println(">>>>> Dispatching to " + msg.target + " " +
                            msg.callback + ": " + msg.what);
                }
                msg.target.dispatchMessage(msg);
                ......
            }
        }
    
- Handler的工作主要包含消息的发送和接收过程


### 流程

a)	Handler发送消息的过程仅仅是向消息队列插入了一条消息

b)	MessageQueue的next方法就会返回这条消息给Looper

c)	Looper收到消息后就开始处理了，最终消息有Looper交由Hander处理
    即Handler的dispatchMessage方法会被调用。这是Handler就进入了处理消息的阶段。

子线程能够通过消息机制更新主线程UI的关键点：
子线程用主线程的Handler发送消息给主线程，更新UI

### 1.	Message Queue

- 队列想象成一个隧道，先进先出FIFO，队列符合先进先出的数据结构
- 队列可以存放很多很多的数据，消息队列的每一数据都是一个消息对象。
 
消息队列主要包含两个操作：插入和读取。

消息队列，由Looper所持有，但是消息的添加是通过Handler进行；　　

消息循环和消息队列都是属于Thread，而Handler本身并不具有Looper和MessageQueue

消息系统的建立和交互，是Thread将Looper和MessageQueue交给某个Handler维护建立消息系统模型。

### 2.	Handler

Handler负责把一个个消息对象加入到消息队列中发送消息派发消息，与此线程消息系统的交互都由Handler完成。

- 一个Thread同时可以对应多个Handler，一个Handler同时只能属于一个Thread。
- Handler属于哪个Thread取决于Handler在那个Thread中建立

消息发送和派发接口：

- post（runnable）消息，Runnable是消息回调，经过消息循环引发消息回调函数执行；
- sendMessage（Message）消息，经过消息循环派发消息处理函数中处理消息；
- dispatchMessage       派发消息，若是post或带有回调函数则执行回调函数，否则执行消息处理函数Handler的handleMessage（通常派生类重写）。

### 3.	Looper

Looper是一个循环器

- 实现Thread的消息循环和消息派发，不停从消息队列中取出消息对象，如果没有对象Looper就阻塞，出于等待的状态。
- 取出消息对象后交给Handler来处理
- 缺省情况下Thread是没有消息循环，需要主动去创建，然后启动Looper的消息循环loop来开始消息循环
- Thread中Looper是唯一的，一个Thread对应一个Looper，。

代码实践

实践一:Work thread 到Main thread消息传递

1.	Main thread中创建Handler对象，实现handleMessage（）方法
2.	在Work thread中通过handler的对象的sendMessage方法把Message发送到MessageQueue里面
3.	再有Looper取出消息，交给Main thread中的handleMessage方法来处理

小结：

1.	Demo说明了Handler，Looper和MessageQueue的价值，之前的那个疑问到这里就解开了。
2.	实现了Work thread到Main thread的消息传递。

Handler在Work thread中发送消息到消息队列，Looper取出消息，调用Handler的HandleMessage在Main thread中进行处理。

这样一个功能非常的重要，因为我们的一些操作像访问网络，大型的IO操作等大多都是在WorkThread中实现。

因为WorkThread不能够直接访问UI的，当我们需要把数据更新到UI上的时候就可以把数据放在Messgae当中发送，然后到MainThread当中去处理然后更新到UI上。

实践二：Main thread到Work thread的消息传递

1.	创建Thread，在Thread中创建handler，准备Looper
2.	MainThread通过handler给Thread发信

注意事项

a)	创建Thread后需要创建Looper对象才能使用Handler
    
   若Looper对象没有创建，就会抛异常
   
   为什么在主线程中不会报错，而在自己新建的线程中就会报这个错误呢？
   
   很简单，因为主线程它已经建立了Looper，你可以打开ActivityThread的源码看一下
    
b)	程序退出时及时清理未处理消息


[github Code](https://github.com/vivianking6855/android-advanced)



<br/>
<br/>


> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)
