---
layout: post
title: 2016-12-19 Android 之 微技巧 （五）
date: 2016-12-19
excerpt: "EventBus, RecyclerView"
tags: [Android]
comments: true
---


## 1. EventBus （解除高度耦合的通讯框架）

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

参考资料：

[快速Android开发系列通信篇之EventBus](http://www.cnblogs.com/angeldevil/p/3715934.html)

[Guava学习笔记：EventBus](http://www.cnblogs.com/peida/p/EventBus.html)

## 2. RecyclerView

RecylerView相对于ListView的优点：

1. RecylerView封装了viewholder的回收复用，也就是说RecylerView标准化了ViewHolder，编写Adapter面向的是ViewHolder而不再是View了，复用的逻辑被封装了，写起来更加简单。
2. 提供了一种插拔式的体验，高度的解耦，异常的灵活，针对一个Item的显示RecylerView专门抽取出了相应的类，来控制Item的显示，使其的扩展性非常强。

    例如：你想控制横向或者纵向滑动列表效果可以通过LinearLayoutManager这个类来进行控制
    
    (与GridView效果对应的是GridLayoutManager,与瀑布流对应的还有StaggeredGridLayoutManager等)，
    
    也就是说RecylerView不再拘泥于ListView的线性展示方式，它也可以实现GridView的效果等多种效果。
    
    你想控制Item的分隔线，可以通过继承RecylerView的ItemDecoration这个类，然后针对自己的业务需求去抒写代码。

3. 可以控制Item增删的动画，可以通过ItemAnimator这个类进行控制，当然针对增删的动画，RecylerView有其自己默认的实现。


如果想使用RecyclerView，需要做以下操作：

- RecyclerView.Adapter - 处理数据集合并负责绑定视图
- ViewHolder - 持有所有的用于绑定数据或者需要操作的View
- LayoutManager - 负责摆放视图等相关操作
- ItemDecoration - 负责绘制Item附近的分割线
- ItemAnimator - 为Item的一般操作添加动画效果，如，增删条目等

[Github Code](https://github.com/vivianking6855/android-ui/tree/ui-advanced)

### Reference

[RecyclerView技术栈](http://www.jianshu.com/p/16712681731e)

[RecyclerView源码分析](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0307/4032.html)

[Android RecyclerView 使用完全解析 体验艺术般的控件](http://blog.csdn.net/lmj623565791/article/details/45059587)

[A First Glance at Android’s RecyclerView](https://www.grokkingandroid.com/first-glance-androids-recyclerview/)

[真实项目运用-RecyclerView封装](http://blog.csdn.net/u014315849/article/details/52537700)

### 开源库

- [SectionedRecyclerViewAdapter](https://github.com/luizgrp/SectionedRecyclerViewAdapter)
- [RecyclerViewItemAnimators](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)


## 3. [DiffUtil](http://blog.csdn.net/zxt0601/article/details/52562770) support-v7:24.2.0的新工具类

DiffUtil可以提高RecyclerView的差异化效率

用来比较两个数据集，寻找出旧数据集和新数据集的最小变化量。它最大的用处就是在RecyclerView刷新时，搭配payload实现高效刷新。 

- [讲解Demo](https://github.com/mcxtzhang/DiffUtils)   
- [效果展示](http://www.itwendao.com/article/detail/51690.html)  
- [使用 Payload 提高 RecyclerView 渲染效率](http://www.tuicool.com/articles/EnyARvQ)

[Github Code](https://github.com/vivianking6855/android-ui/tree/ui-advanced)

