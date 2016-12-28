---
layout: post
title: 2016-12-28 Android 之 微技巧 （七）M new feature 2
date: 2016-12-28
excerpt: "CoordinatorLayout, AppBarLayout, CollapsingToolbarLayout"
tags: [Android]
comments: true
---

## 1. CoordinatorLayout ：super-powered FrameLayout

[android CoordinatorLayout使用](http://blog.csdn.net/xyz_lmn/article/details/48055919)

[Github Code](https://github.com/vivianking6855/android-ui/tree/ui-advanced)


- CoodinatorLayout, 从名字上可以看就是帮我们协调子View的。根据我们的定制，帮助我们协调各个子View的布局。
- 和传统的ViewGroup不同，子view从此知道了彼此之间的存在，一个子view的变化可以通知到另一个子view。
- CoordinatorLayout所做的事情就是当成一个通信的桥梁，连接不同的view。
- CoordinatorLayout的使用核心是Behavior.

### Reference

[CoordinatorLayoutDemo](https://github.com/ffuujian/CoordinatorLayoutDemo)

[Android官方文档](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html)

[CoordinatorLayout 与 Behaviors 初探](https://segmentfault.com/a/1190000002888109)

[CoordinatorLayout的使用如此简单](http://www.jianshu.com/p/72d45d1f7d55)

## 2. 玩转AppBarLayout

[玩转AppBarLayout，更酷炫的顶部栏 ](http://www.jianshu.com/p/d159f0176576)


