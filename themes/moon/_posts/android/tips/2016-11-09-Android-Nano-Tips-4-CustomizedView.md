---
layout: post
title: 2016-11-09 UI - Customized View
date: 2016-11-09
excerpt: "Customized View"
tags: [Android 微技巧]
comments: true
---


## Customized View : 自定义View

### 为什么需要自定义View

1.	现有的View满足不了你的需求，也没有办法从已有控件派生一个出来；界面元素需要自己绘制。
2.	现有View可以满足要求，把它做成自定义View只是为了抽象

为这个自定义View提供若干方法，方便调用着操纵View。

通常做法是派生一个已有View，或者结合xml文件直接inflate。

### Attention

能够用Android基础控件解决的问题就尽量用基础控件，其次是用基础控件的组合。如果是确实有必要自定义才考虑自定义。

自定义的控件，既需要耗费较长的开发时间，又不一定能保证有基础控件那么高的效率（基础控件都是谷歌优化过了的）。

### 如何系统学习自定义控件

推荐资料

1.	Google Doc : https://developer.android.com/training/custom-views/index.html 
2.	[Android技术专题]自定义View : https://zhuanlan.zhihu.com/p/21995633
3.	自定义View系列教程  : http://blog.csdn.net/lfdfhl/article/details/51671038

### 关键点

1.	自定义控件的价值和使用场景
2.	三大方法： onMeasure，onLayout，Draw
3.	自定义属性
4.	事件传递

### 应用场景

- 组合控件：试题控件（TextView+VideoGroup）、下拉刷新、瀑布流控件、带左/右滑功能的控件、视频控件等。

    通过Android的基础控件(TextView、CheckBox、Button、ProgressBar等)组合而成，

- 完全自定义控件：继承自View、TextureView或SurfaceView，然后重写核心的回调方法。

    比如：webview + loading动画（SurfaceView）、
    
    比如输入法中的手写控件、图文混排控件（现在很多都是通过webview加载网页实现了）、词典取词控件、图表控件、个性化进度条、
    
    弹幕显示控件、Markdown控件、IDE代码编辑控件等。

### 自定义View Class & 属性实践

How to create Customized View and 属性”key”. 

参看[Google create-view](https://developer.android.com/training/custom-views/create-view.html)

### 自定义View项目实践

[Github Code](https://github.com/vivianking6855/android-ui/tree/ui-basic)


### 优质开源库

- awesome-android-ui : https://github.com/wasabeef/awesome-android-ui 
- android-open-project : https://github.com/Trinea/android-open-project 


### Reference


> [如何系统学习自定义控件 ](https://zhuanlan.zhihu.com/p/21995633)

> [Android技术专题自定义View](https://www.zhihu.com/question/41101031)

> [推翻自己和过往，重学自定义View](https://link.zhihu.com/?target=http%3A//blog.csdn.net/lfdfhl/article/details/51671038)

> [深入了解View](https://link.zhihu.com/?target=http%3A//blog.csdn.net/guolin_blog/article/details/12921889)

> [Android 深入理解Android中的自定义属性](https://link.zhihu.com/?target=http%3A//blog.csdn.net/lmj623565791/article/details/45022631)
