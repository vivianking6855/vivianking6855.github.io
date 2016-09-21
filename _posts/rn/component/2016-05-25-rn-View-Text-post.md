---
layout: post
title: ReactNative 学习笔记 Component - View,Text
date: 2016-05-27
excerpt: "ReactNative 学习笔记 Component - View,Text"
tags: [ReactNative,学习笔记]
comments: true
---

## ReactNative 学习笔记 6 Component - View,Text

### View

[跟标签div类似,支持多层嵌套，支持flexbox布局](http://reactnative.cn/docs/0.26/view.html#content)

实例布局逻辑分析：

1. 水平均分成三栏
2. 左边部分内容和布局都居中
3. 中间部分内容和布局都居中；“中上”和“中下” 区域左右两边有竖线。“中上”，“中下”均分中间部分，中间有横线
4. 右边部分内容和布局都居中；“右上”和“右下”均分右边部分，中间有横线


![View Demo](http://i.imgur.com/e6IN6Lh.png)

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/ViewLesson.js)


### Text
[用于显示文本。可以嵌套，继承。](http://reactnative.cn/docs/0.26/text.html#content)

Text组件特性：

1. onPress 当文本被点击以后调用此回调函数。
2. numberOfLines 最多显示行数
3. onLayout function 当挂载或者布局变化以后调用

实例布局逻辑分析：
1. 单独的module header，便于以后重复使用。其他控件也可参照处理
2. 自定义中间List
3. 自定义ImportNews 

![TextDemo](http://i.imgur.com/oOUUx5i.png)


[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/TextLesson.js)

> [key warning ](http://facebook.github.io/react/docs/multiple-components.html#dynamic-children)