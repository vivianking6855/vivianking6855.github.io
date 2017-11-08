---
layout: post
title: IdleHandler
date: 2017-11-8
excerpt: "IdleHandler"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

 IdleHandler是MessageQueue中的一个Interface，在looper里面的message暂时处理完了，这个时候会回调这个接口
 
 返回false，那么调用完成后就会移除它，返回true就会在下次message处理完了的时候继续回调。
 
 因为它的特别的调用时机：looper里面的message暂时处理完了调用，因此有很多不错的用途

# 源码

IdleHandler在源码中的定义


    public final class MessageQueue {
    
        Message mMessages;
        private final ArrayList<IdleHandler> mIdleHandlers = new ArrayList<IdleHandler>();



        /**
         * Callback interface for discovering when a thread is going to block
         * waiting for more messages.
         */
        public static interface IdleHandler {
            /**
             * Called when the message queue has run out of messages and will now
             * wait for more.  Return true to keep your idle handler active, false
             * to have it removed.  This may be called if there are still messages
             * pending in the queue, but they are all scheduled to be dispatched
             * after the current time.
             */
            boolean queueIdle();
        }
        
# IdleHandler的妙用    

## 1. 新的生命周期回调时机：绘制完成的回调

如果想要在某个activity绘制完成去做一些事情，那么就可以用到IdleHandler

以前我们有在onResume()的时候做，onResume代表用户可交互，但实际上onResume不带条绘制完成

看下图（摘录自腾讯Bugly)，android那些耗时的measure, layout, draw都是在performTraversals函数中执行的。

![调用流程](https://mmbiz.qpic.cn/mmbiz_jpg/tnZGrhTk4debqCZSonTg6eu3Vmf02hFfIkenlmVibkGCCibRptkTIxqkFC3fK1rcmw5ULlCqRLvjjPgLFZJdTbNw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

所以如果我们想在界面绘制出来后做什么，在onResume里面不太合适，因为它先于measure等流程。

如果是在onResume里面post一个message，也是在绘制之前

![调用流程](https://mmbiz.qpic.cn/mmbiz_jpg/tnZGrhTk4debqCZSonTg6eu3Vmf02hFfdkOQE9OpO6cwyuuHHqVaAXQak2KW5hMToia7icS8n4XicgtsrqrmJ87CQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)


如果是用IdleHandler的话，流程如下：

![IdleHandler调用流程](https://mmbiz.qpic.cn/mmbiz_jpg/tnZGrhTk4debqCZSonTg6eu3Vmf02hFfdkOQE9OpO6cwyuuHHqVaAXQak2KW5hMToia7icS8n4XicgtsrqrmJ87CQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

    Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
                @Override
                public boolean queueIdle() { 
                   // do your job here
              
                    return false; // 返回false，那么调用完成后就会移除它
                }
            });

如果是同一个页面加载多个复杂的layout，而且有显示先后要求的话可以试试用这个

## 2. 可以结合HandlerThread, 用于单线程消息通知器

比如有一个HandlerThread负责数据的管理：增删改查等，如果有非常频繁的增删改查，我们又不希望监听的对象拿到如此频繁的通知。

那么我们可以在频繁操作的时候把DataChange存起来，在IdleHandler的时机时在进行通知。

    public DataModel() {        
          if (sThread == null) {
                sThread = new HandlerThread("data-model");
                sThread.start();
            }
            mHandler = new Handler(sThread.getLooper());        
          try {
                MessageQueue queue = sThread.getLooper().getQueue();
                queue.addIdleHandler(new MessageQueue.IdleHandler() {                
    
                   @Override
                   public boolean queueIdle() {                    
                   if (mListeners != null){                        
                        for (DataChangeListener<T> mListener : mListeners) {
                            mListener.onDataChange(new ArrayList<>(mData));
                         }
                    }                    
                   return true; // 代表长期监听
               } });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        
        
注意我们这要用HandlerThread的Looper来获取MessageQueue. IdleHandler返回了true, 表示要长期监听消息队列。


# 总结

IdleHandler，从命名就可以看出是队列为空时Handler，即在looper里面message暂时执行完毕了就会回调。非常好的一个绘制完成的回调时机。

不过如果有些数据用户如果要在view渲染前获取，用这个就不合适了。这类需求常用的做法是UI上设置等待或广告页，后台单开Thread默默加载完成，通知页面渲染。

IdleHandler的空闲时机也可以用在省去监听数据变化时，频繁操作的频繁通知。仅在idle的进行通知。

# Reference

- [腾讯Bugly：你知道android的MessageQueue.IdleHandler吗](https://mp.weixin.qq.com/s/KpeBqIEYeOzt_frANoGuSg) 
