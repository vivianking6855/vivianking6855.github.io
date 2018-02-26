---
layout: post
title: 开源库 - EventBus
date: 2016-12-19
excerpt: "EventBus"
categories: Android
tags: [Android 进阶]
comments: true
---


# EventBus （解除高度耦合的通讯框架）

[EventBus框架： 一个典型的发布者-订阅者模式](https://github.com/greenrobot/EventBus )

跨不同的类去调用方法，耦合度很高。另外listener的方式在需要大量通知的时候回带来接口的膨胀。

在这些情境下，建议使用EventBus.它的核心思想：

a. 首先定义自己的Event(其实是随便写个类)

例子：Headline需传一个点击时候的listview的position给Detail fragment. 

可以写个包装int类型的MessageEvent:
   
    public class MessageEvent {
    
        public int position;
    
        public MessageEvent(int position) {
            this.position = position;
        }
    }
    
b.  在订阅者（即Detail fragment或者说FragmentB）中，onstart和onstop要注册/解注册：

    //DetailFragment.java
    
    @Override
    public void onStart() {
        super.onStart();
        EventBus.getDefault().register(this);
    }
    
    @Override
    public void onStop() {
        super.onStop();
        EventBus.getDefault().unregister(this);
    }

c. 在订阅者（即Detail fragment或者说FragmentB）中，写一个名字叫onMessageEvent的函数。

带回来的参数是MessageEvent类型，这个函数即被调用的方法，等待Headline传过来包装了 position数据的MessageEvent对象

    //DetailFragment.java
        
    @Subscribe(threadMode = ThreadMode.MAIN)  
    public void onMessageEvent(MessageEvent event){
        int index = event.positioin;
       // do something using index,比如刷新文章内容
    }

d. 在发布者（即Headline Fragment或者说Fragment A）中，在合适的地方执行post方法。

即发布出去这个MessageEvent，这里即在HeadLineFragment的listview被点击的回调中发送post:

    //FragmentA.java
    
    @Override 
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {        
      EventBus.getDefault().post(new MessageEvent(position));
    }


这样子，DetailFragment的onEvent方法就会被调用。

含有position的MessageEvent也就被传了过去，随之DetailFragment根据拿到的position来刷新界面。

两个fragment没有写任何的interface也没implements任何的接口，也不用实例化一个listenser并且执行所谓的setonlistener。

内部流程：

1. EventBus在某个组件中注册的时候回记录下这个组件里面的onMessageEvent函数
2. 一个应用中可能有多个onMessageEvent函数等待被调用，这里即通过onMessageEvent函数的参数类型和个数来区别。

注意事项：

EventBus是通过java反射机制来找到这个onEvent方法的，所以打包的时候若需要混淆的话千万记住不要混淆onMessageEvent这个函数。

可以在proguard文件中添加：

    -keepclassmembers class ** {
        public void onMessageEvent*(**);
    }
    
    
# 其他

当然EventBus在解除我们耦合的同时，也有很多的潜在风险

这两篇文章有提到建议停止使用EventBus

[为什么你应该停止使用EventBus](http://blog.csdn.net/mislead/article/details/46695869)

[Endless While Loop](https://endlesswhileloop.com/blog/2015/06/11/stop-using-event-buses/)

建议使用用[RXJava](http://gank.io/post/560e15be2dca930e00da1083)[替代EventBus](https://github.com/kaushikgopal/RxJava-Android-Samples)进行事件的分发。

# 参考资料：

[快速Android开发系列通信篇之EventBus](http://www.cnblogs.com/angeldevil/p/3715934.html)

[Guava学习笔记：EventBus](http://www.cnblogs.com/peida/p/EventBus.html)
