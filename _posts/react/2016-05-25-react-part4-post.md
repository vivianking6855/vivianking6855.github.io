---
layout: post
title: ReactNative
date: 2015-05-25
excerpt: "ReactNative 学习笔记 4"
tags: [react]
comments: true
---


## ReactNative  第 8 节 JSX实战
React Native中没有DOM的概念，只有组件的概念。<br/>
因此在ReactJS中使用的Html标签以及对DOM的操作是不起作用的。<br/>
不过组件的生命周期、JSX的语法、事件绑定、自定义属性等，在RN和ReactJS中是一样的。

用组件嵌套组件的方式实现，效果图：<br/>
![效果图](http://i.imgur.com/RlUC68h.jpg)

其中的大概逻辑是：

MargginBox ：使用Box，childName="borderBox"，通过{this.props.children}把内部子组件传给子组件Box

BorderBox ：使用Box，childName="paddingBox"，通过{this.props.children}把内部子组件传给子组件Box

PaddingBox ：使用Box，childName="elementBox"，通过{this.props.children}把内部子组件传给子组件Box

ElementBox ：最内层的组件

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/master/TwoReactNative/index.android.js)