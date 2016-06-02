---
layout: post
title: ReactNative
date: 2015-05-27
excerpt: "ReactNative 学习笔记 6"
tags: [react]
comments: true
---

## ReactNative 第 10 节 组件

### View

跟标签div类似,支持多层嵌套，支持flexbox布局

布局逻辑分析：

1. 水平均分成三栏
2. 左边部分内容和布局都居中
3. 中间部分内容和布局都居中；“中上”和“中下” 区域左右两边有竖线。“中上”，“中下”均分中间部分，中间有横线
4. 右边部分内容和布局都居中；“右上”和“右下”均分右边部分，中间有横线


![View Demo](http://i.imgur.com/e6IN6Lh.png)

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/ViewLesson.js)

### 