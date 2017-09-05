---
layout: post
title: UI - BottomNavigationView 
date: 2017-9-1
excerpt: "BottomNavigationView "
categories: Android
tags: [Android 基础]
comments: true
---

# 简介

Material Design 很早就有了BottomNavigationView类，不过知道Support Library 25 才增加了BottomNavigationView控件。 

# 基本用法

activity_main.xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context="com.open.bottomnav.MainActivity">
    
        <FrameLayout
            android:id="@+id/content"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1">
    
            <TextView
                android:id="@+id/message"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="@dimen/activity_vertical_margin"
                android:layout_marginLeft="@dimen/activity_horizontal_margin"
                android:layout_marginRight="@dimen/activity_horizontal_margin"
                android:layout_marginTop="@dimen/activity_vertical_margin"
                android:text="@string/title_home" />
    
        </FrameLayout>
    
        <android.support.design.widget.BottomNavigationView
            android:id="@+id/navigation"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom"
            android:background="?android:attr/windowBackground"
            app:itemBackground="@color/itemBackground"
            app:itemIconTint="@drawable/bottom_icon_tint"
            app:itemTextColor="@drawable/bottom_icon_tint"
            app:menu="@menu/navigation" />
    
    </LinearLayout>

BottomNavigationView的其他属性：

- app:itemIconTint 设置图标的颜色。可以依据state_checked设定normal和check状态的颜色
- app:itemTextColor 设置文字的颜色。。可以依据state_checked设定normal和check状态的颜色
- app:itemBackground 设置整个BottomNavigationView的背景色。跟android:background效果一样。


menu -> navigation.xml

    <?xml version="1.0" encoding="utf-8"?>
    <menu xmlns:android="http://schemas.android.com/apk/res/android">
    
        <item
            android:id="@+id/navigation_home"
            android:icon="@drawable/ic_home_black_24dp"
            android:title="@string/title_home" />
    
        <item
            android:id="@+id/navigation_dashboard"
            android:icon="@drawable/ic_dashboard_black_24dp"
            android:title="@string/title_dashboard" />
    
        <item
            android:id="@+id/navigation_notifications"
            android:icon="@drawable/ic_notifications_black_24dp"
            android:title="@string/title_notifications" />
    
    </menu>

MainAcitivy.java

    package com.open.bottomnav;
    
    import android.os.Bundle;
    import android.support.annotation.NonNull;
    import android.support.design.widget.BottomNavigationView;
    import android.support.v7.app.AppCompatActivity;
    import android.view.MenuItem;
    import android.widget.TextView;
    
    public class MainActivity extends AppCompatActivity {
    
        private TextView mTextMessage;
    
        private BottomNavigationView.OnNavigationItemSelectedListener mOnNavigationItemSelectedListener
                = new BottomNavigationView.OnNavigationItemSelectedListener() {
    
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                switch (item.getItemId()) {
                    case R.id.navigation_home:
                        mTextMessage.setText(R.string.title_home);
                        return true;
                    case R.id.navigation_dashboard:
                        mTextMessage.setText(R.string.title_dashboard);
                        return true;
                    case R.id.navigation_notifications:
                        mTextMessage.setText(R.string.title_notifications);
                        return true;
                }
                return false;
            }
    
        };
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mTextMessage = (TextView) findViewById(R.id.message);
            BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.navigation);
            navigation.setOnNavigationItemSelectedListener(mOnNavigationItemSelectedListener);
        }
    
    }

Android Studio创建Activity时选择就可以自动生成上面的Code：

![](http://i.imgur.com/CIY01UA.jpg)

# 如何去掉BottomNavigationView的动画效果

不过当item个数大于三个的时候，会出现不一样的效果：一会儿带文字，一会儿不带文字，时高时低，时大时小

这在大陆算是较为奇怪的体验，特别是大陆很多app都是4个bottom icon.

![](http://i.imgur.com/ZxFFYUm.gif)

遗憾的是官方没有自定义的方法可以改变这种默认的行为，现在比较通用的方法是通过反射的方式

Code：

BottomNavigationViewHelper.java

    public class BottomNavigationViewHelper {
        private final static String TAG = "BottomNavHelper";
    
        public static void disableShiftMode(BottomNavigationView view) {
            final String CLASS_FIELD = "mShiftingMode";
            BottomNavigationMenuView menuView = (BottomNavigationMenuView) view.getChildAt(0);
            try {
                Field shiftingMode = menuView.getClass().getDeclaredField(CLASS_FIELD);
                shiftingMode.setAccessible(true);
                shiftingMode.setBoolean(menuView, false);
                shiftingMode.setAccessible(false);
                for (int i = 0; i < menuView.getChildCount(); i++) {
                    BottomNavigationItemView item = (BottomNavigationItemView) menuView.getChildAt(i);
                    //noinspection RestrictedApi
                    item.setShiftingMode(false);
                    // set once again checked value, so view will be updated
                    //noinspection RestrictedApi
                    item.setChecked(item.getItemData().isChecked());
                }
            } catch (NoSuchFieldException e) {
                Log.d(TAG, "Unable to get shift mode field", e);
            } catch (IllegalAccessException e) {
                Log.d(TAG, "Unable to change value of shift mode", e);
            }
        }
    }

MainActivity.java

        BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.navigation);
        BottomNavigationViewHelper.disableShiftMode(navigation);

# 注意事项

- BottomNavigationView的默认高度为56dp
- 官方建议菜单项不多于5个
- 为了方便设置图标的颜色,建议使用SVG图片（Scalable Vector Graphics矢量图)

# Reference

[如何去掉BottomNavigationView的动画效果](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2017/0428/7888.html)

[底部导航栏BottomNavigationView系统库与第三方库的两种不同库的使用方法](http://blog.csdn.net/huangxiaoguo1/article/details/53842536)

