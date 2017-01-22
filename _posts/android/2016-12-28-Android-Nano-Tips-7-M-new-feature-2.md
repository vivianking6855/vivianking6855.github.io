---
layout: post
title: 2016-12-28 Android 之 微技巧 （七）M new feature 2
date: 2016-12-28
excerpt: "CoordinatorLayout, AppBarLayout, CollapsingToolbarLayout，NestedScrollView"
tags: [Android]
comments: true
---

## 1. CoordinatorLayout ：super-powered FrameLayout

CoodinatorLayout 从名字上可以看就是帮我们协调子View的，一个通信的桥梁，连接不同的view。

CoordinatorLayout的使用核心是Behavior.

### 一个最简单的实践

最简单的实践包括：

- Dependency，依赖：来控制其他View
- Child依据Dependency来做事情，Child设定Behavior
- [Behavior](http://www.jianshu.com/p/a506ee4afecb)：当Dependency变化 (position, width, height)时，通过Behavior连接Child

        -----------Behavior的两个核心方法
        /**
        * 判断child的布局是否依赖dependency
        */
           @Override
         public boolean layoutDependsOn(CoordinatorLayout parent, T child, View dependency) {
            boolean rs;
            //根据逻辑判断rs的取值
            //返回false表示child不依赖dependency，ture表示依赖
            return rs;    
        }
        
        /**
        * 当dependency发生改变时（位置、宽高等），执行这个函数
        * 返回true表示child的位置或者是宽高要发生改变，否则就返回false
        */
        @Override
        public boolean onDependentViewChanged(CoordinatorLayout parent, T child, View dependency) {
             //child要执行的具体动作
                return true;
        }

### 触发流程

Dependency Change -> 触发Behavior onDependentViewChanged -> Child Change

当然还有一个依赖检查：layoutDependsOn，判断Child是否依赖Dependency

### 核心Code

        -----------Behavior两个方法实现
        /**
         * check if child layout depend on dependency view
         */
        @Override
        public boolean layoutDependsOn(CoordinatorLayout parent, TextView child, View dependency) {
            // child is the view which set this Behavior
            // if dependency is instance of CoordinatorStartView, means it is the dependency that we need
            return dependency instanceof CoordinatorDependencyView;
        }
    
        /**
         * when dependency changed (position, width, height), call this method
         * if true, means child's position, width, height will change
         */
        @Override
        public boolean onDependentViewChanged(CoordinatorLayout parent, TextView child, View dependency) {
            // set child(TextView) according to dependency
            int left = dependency.getLeft();
            int top = dependency.getTop();
    
            int x = screenWidth - left - child.getWidth();
            int y = top;
    
            setPosition(child, x, y);
            return true;
        }
    
        /**
         * @param child child view that will change position
         * @param x     leftMargin
         * @param y     topMargin
         * @ description set position of child
         */
        private void setPosition(View child, int x, int y) {
            if (child.getLayoutParams() instanceof CoordinatorLayout.LayoutParams) {
                CoordinatorLayout.LayoutParams params = (CoordinatorLayout.LayoutParams) child.getLayoutParams();
                params.leftMargin = x;
                params.topMargin = y;
                child.setLayoutParams(params);
            }
        }
        
        
        ----------- layout
        
        <?xml version="1.0" encoding="utf-8"?>
        <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        
        <TextView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="@android:color/holo_red_light"
            android:text="@string/child"
            app:layout_behavior="com.ui.advanced.advanceddemo.coordinatorlayout.start.UserBehavior" />
        <!--app:layout_behavior=" your Behavior class name,include package name"-->
        <!--CoordinatorLayout will reflect your Behavior-->
        
        <com.ui.advanced.advanceddemo.coordinatorlayout.start.DependencyView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="@android:color/darker_gray"
            android:text="@string/dependency" />
        
        </android.support.design.widget.CoordinatorLayout>




[Github Code](https://github.com/vivianking6855/android-ui/tree/ui-advanced/AdvancedDemo/app/src/main/java/com/ui/advanced/advanceddemo/coordinatorlayout/start)


## 2. 玩转AppBarLayout

AppBarLayout继承自LinearLayout，布局方向为垂直方向。所以你可以把它当成垂直布局的LinearLayout来使用。

AppBarLayout可以定制当某个可滚动View的滚动手势发生变化时，其内部的子View实现何种动作。

核心属性：app:layout_scrollFlags。 如果View想要滚动出屏幕需要设置这个flag。没有设置这个flag的view将被固定在屏幕顶部。

核心Code:

    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/activity_app_bar_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
        tools:context="com.ui.advanced.advanceddemo.appbarlayout.AppBarLayoutActivity">
    
        <android.support.design.widget.AppBarLayout
            android:id="@+id/appbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:theme="@style/AppTheme.AppBarOverlay">
    
            <!--scroll: 所有想滚动出屏幕的view都需要设置这个flag，没有设置这个flag的view将被固定在屏幕顶部。
                例如，TabLayout 没有设置这个值，将会停留在屏幕顶部-->
            <!--enterAlways: 设置这个flag时，向下的滚动都会导致该view变为可见，启用快速“返回模式”-->
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                app:layout_scrollFlags="scroll|enterAlways"
                app:logo="@drawable/ic_star"
                app:popupTheme="@style/AppTheme.PopupOverlay"
                app:subtitle="@string/tool_bar_sub_title"
                app:title="@string/app_bar"
                app:titleMarginStart="30dp" />
    
            <!-- if 当ViewPager过多展示不开，需要滚动时设置app:tabMode="scrollable"-->
            <!--否则默认不需要设置，默认为app:tabMode="fixed"，ViewPager平分宽度-->
            <android.support.design.widget.TabLayout
                android:id="@+id/tab"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:tabMode="scrollable" />
    
        </android.support.design.widget.AppBarLayout>
    
        <!--"@string/appbar_scrolling_view_behavior"系统的-->
        <!--不加app:layout_behavior="@string/appbar_scrolling_view_behavior"
            会导致RecyclerView在Toolbar下面，被TabLayout遮盖-->
        <android.support.v4.view.ViewPager
            android:id="@+id/viewpager"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior" />
    
    </android.support.design.widget.CoordinatorLayout>


[玩转AppBarLayout，更酷炫的顶部栏 ](http://www.jianshu.com/p/d159f0176576)


CoordinatorLayout,AppBarLayout,CollapsingToolbarLayout,NestedScrollView搭配起来可以实现下面的效果

![](http://i.imgur.com/vP6PGiO.gif)

<br>


> [CoordinatorLayout的使用如此简单](http://www.jianshu.com/p/72d45d1f7d55)

> [CoordinatorLayoutDemo](https://github.com/ffuujian/CoordinatorLayoutDemo)

> [android CoordinatorLayout使用](http://blog.csdn.net/xyz_lmn/article/details/48055919)

> [Android官方文档](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html)

> [CoordinatorLayout 与 Behaviors 初探](https://segmentfault.com/a/1190000002888109)

> [Android M新控件之FloatingActionButton，TextInputLayout，Snackbar，TabLayout的使用](http://blog.csdn.net/feiduclear_up/article/details/46500865)

> [Android M新控件之AppBarLayout，NavigationView，CoordinatorLayout，CollapsingToolbarLayout的使用 ](http://blog.csdn.net/feiduclear_up/article/details/46514791)

> [DesignSupportLibraryExamples](https://github.com/PareshMayani/DesignSupportLibraryExamples)



