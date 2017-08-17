---
layout: post
title: Android Fragment
date: 2017-8-17
excerpt: "Android Fragment"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

Fragment可以当成Activity的一个界面的一个组成部分，Activity的界面可以完全有不同的Fragment组成

Fragment拥有自己的生命周期和接收、处理用户的事件。可以动态的添加、替换和移除某个Fragment。

# 1. Fragments and UI Modularization

Fragments实现UI 模块化布局

activity_main.xml
    
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <!-- List of Book Titles -->
        <fragment
            android:id="@+id/fragmentTitles"
            android:name="com.open.dynamicfragments.BookListFragment"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />
    
        <!-- Description of selected book -->
        <fragment
            android:id="@+id/fragmentDescription"
            android:name="com.open.dynamicfragments.BookDescFragment"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />
    </LinearLayout>


fragment_book_desc.xml

    <?xml version="1.0" encoding="utf-8"?>
    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/scrollDescription"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="fill_horizontal"
            android:text="@string/dynamicUiDescription" />
    </ScrollView>


fragment_book_list.xml
    
    <?xml version="1.0" encoding="utf-8"?>
    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/scrollTitles"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
        <RadioGroup
            android:id="@+id/bookSelectGroup"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
    
            <RadioButton
                android:id="@+id/dynamicUiBook"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:checked="true"
                android:text="@string/dynamicUiTitle" />
    
            <RadioButton
                android:id="@+id/android4NewBook"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/android4NewTitle" />
            <!-- Other RadioButtons elided for clarify -->
        </RadioGroup>
    </ScrollView>

[完整Code](https://github.com/vivianking6855/android-advanced/tree/master/DynamicFragments/DynamicFragments-Chapter1)

# 2. Fragments and UI Flexibility

## Fragment实现动态布局

功能需求

- 横屏选择书单，右侧内容变化
- 竖屏选择书单，开启另外一个Activity

实现逻辑：

- 横屏main layout包含两个Fragment（BookDescFragment，BookListFragment）在MainActivity中
- 竖屏main layout一个Fragment（BookListFragment）在MainActivity中；BookDescFragment在另外一个BookDescActivity
- 判断逻辑：如果BookDescFragment存在main layout中，则更新右侧内容，否则叫起另一个Activity

values-land家中使用refs.xml layout alias方式实现

    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <item name="activity_main" type="layout">
            @layout/activity_main_wide
        </item>
    </resources>


## Fragment间交互

- BookListFragment定义Listener
- MainActivity中监听，然后调用BookDescFragment中的方法。

[完整Code](https://github.com/vivianking6855/android-advanced/tree/master/DynamicFragments/DynamicFragments-Chapter2)

# 3. 生命周期

[Activity 生命周期](https://developer.android.com/reference/android/app/Activity.html#ActivityLifecycle)

Fragment 生命周期

![](http://i.imgur.com/h61e5oE.jpg)

![](http://i.imgur.com/PqQfmUA.jpg)


# Reference


> [Creating Dynamic UI with Android Fragments](https://github.com/vivianking6855/android-advanced/blob/master/DynamicFragments/Creating%20Dynamic%20UI%20with%20Android%20Fragments.pdf)
