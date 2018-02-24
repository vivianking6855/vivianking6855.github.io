---
layout: post
title: RecyclerView封装 四
date: 2017-10-23
excerpt: "RecyclerView封装 四"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 封装四、 下拉刷新

## 1. 用SwipeRefreshLayout实现

xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
    
        <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/swipe_refresh"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
    
            <android.support.v7.widget.RecyclerView
                android:id="@+id/recycler_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent" />
    
        </android.support.v4.widget.SwipeRefreshLayout>
    </LinearLayout>


创建一个SwipeRefreshLayout对象

     mRefreshLayout = (SwipeRefreshLayout)findViewById(R.id.swipe_refresh);
    
添加OnRefreshListener监听：

    mRefreshLayout.setOnRefreshListener(new OnRefreshListener(){
        public void onRefresh() {
            // load data and refresh ui here
        }
    });
    
在数据加载完成后，调用mRefreshLayout.setRefreshing(false) 加载icon

效果图
 
![](https://i.imgur.com/heGhSkj.jpg)


## 2. 基于三方库实现

上面效果不是那么炫酷，而且跟主流的刷新效果不太一样。

发现有很多好用的库，比如[CommonPullToRefresh](https://github.com/Chanven/CommonPullToRefresh)，[BGARefreshLayout](https://github.com/bingoogolapple/BGARefreshLayout-Android) 等

那我们就站在巨人肩膀上，不重复造轮子了


# [RecyclerView封装目录](http://vivianking6855.github.io/2018/02/24/RecyclerView-Advance-index/)
