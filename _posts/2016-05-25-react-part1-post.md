---
layout: post
title: ReactNative
date: 2015-05-25
excerpt: "ReactNative 学习笔记 （一）"
tags: [react, technology]
comments: true
---


## ReactNative 第1节 必备的官网和blog ##
- FaceBook ReactNative官网 ：[https://facebook.github.io/react-native/](https://facebook.github.io/react-native/ "https://facebook.github.io/react-native/") 
- ReactNative中文blog ： [http://reactnative.cn/blog.html](http://reactnative.cn/blog.html "http://reactnative.cn/blog.html")
- React 官网：[http://facebook.github.io/react/index.html](http://facebook.github.io/react/index.html "http://facebook.github.io/react/index.html")
- React 中文：[http://react-china.org/](http://react-china.org/ "http://react-china.org/")
- 手把手教ReactNative：[http://reactnative.cn/post/759](http://reactnative.cn/post/759 "http://reactnative.cn/post/759")
- JS库查询 [https://cdnjs.com/](https://cdnjs.com/ "https://cdnjs.com/")

##  ReactNative 第2节  环境搭建 ##
[https://yunpan.cn/cPnMZrdLHiVu4 ](https://yunpan.cn/cPnMZrdLHiVu4  "ReactNative环境搭建")
（提取码：dc13）


##  ReactNative 第3节 测试ReactNativeProject ##

 1. cmd运行命令行：react-native init AwesomeProject
 2. cd AwesomeProject，cmd运行 react-native start
 3. cmd打开另外一个命令行。cd AwesomeProject，cmd运行react-native run-android

**如果是Device运行时，出现红色的一些提示。可以试试下面的操作**
（Android 5.0的系统）
1. cmd开一个新的命令行，运行 adb reverse tcp:8081 tcp:8081
2. 设备端Reload JS


## Run on a Device ##
adb reverse tcp:8081 tcp:8081

> [http://reactnative.cn/post/759](http://reactnative.cn/post/759)


## Tips ##

### node_modules文件名或扩展名太长如何删除 ##
步骤一：npm install -g rimraf
步骤二：rimraf node_modules