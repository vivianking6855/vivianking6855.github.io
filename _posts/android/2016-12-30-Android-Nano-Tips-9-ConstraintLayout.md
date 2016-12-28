---
layout: post
title: 2016-12-30 Android 之 微技巧 （九）ConstraintLayout
date: 2016-12-30
excerpt: "ConstraintLayout"
tags: [Android]
comments: true
---


## ConstraintLayout（拖动来写布局）

[一步步理解使用 ConstraintLayout](http://www.jianshu.com/p/793f76cf9fea)

### 关键点：

- Constraint Handle 约束手柄
- View Inspector.列出所选控件的属性,对齐方式以及所有的约束.
- 手工约束
- AutoConnect 自动连接 
- Inference	推理

如果不识别包：com.android.support.constraint. 需要到SDK Manager中下载配置

- 进入SDK Tools/Support Repository 勾选"ConstraintLayout for Android" 下载
- 勾选“Show Package Details” 查看下载的版本 （截止blog更新版本是“compile 'com.android.support.constraint:constraint-layout:1.0.0-beta4'”）

[Github Code](https://github.com/vivianking6855/android-ui/tree/ui-advanced)

### Reference

> [了解使用Android ConstraintLayout](http://quanqi.org/2016/05/20/code-labs-constraint-layout/)

> [google constraint-layout](https://codelabs.developers.google.com/codelabs/constraint-layout/index.html#0)

> [google Git Code](https://github.com/googlecodelabs/constraint-layout)
