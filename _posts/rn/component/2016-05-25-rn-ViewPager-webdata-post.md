---
layout: post
title: ReactNative
date: 2015-06-08
excerpt: "ReactNative 学习笔记 Component - ViewPager， 异步网络数据"
tags: [ReactNative]
comments: true
---

## Component - ViewPager

一个允许在子视图之间左右翻页的容器。每一个ViewPagerAndroid的子容器会被视作一个单独的页，并且会被拉伸填满ViewPagerAndroid。

注意所有的子视图都必须是纯View，而不能是自定义的复合容器。你可以给每个子视图设置样式属性譬如padding或backgroundColor。


关键代码：
            <View style={[styles.flex,{marginTop:45}]}>
                <ProgressBarAndroid styleAttr="LargeInverse" />
           </View>
   


### third-party lib

有很多开源的库支持左右翻页


[1. React-Native-ViewPager](https://github.com/zbtang/React-Native-ViewPager)

ViewPager and Indicator component for react-native on both android and ios. 

ViewPager's props is the same as ViewPagerAndroid.

[2. react-native-scrollable-tab-view](https://github.com/skv-headless/react-native-scrollable-tab-view)

This is probably my favorite navigation pattern on Android, 

I wish it were more common on iOS! This is a very simple JavaScript-only implementation of it for React Native.

For more information about how the animations behind this work, check out the Rebound section of the React Native Animation Guide


[Sample Code ViewPager](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/ViewPagerLesson.js)

[Sample Code GuidePage](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/project1-GuidePage/GuidePage.js)



## 异步网络数据

ReactNative不同于android的handler，AsyncTask。他是通过this.state 来触发UI更新。

RN中的网络请求：XMLHttpRequest， Fetch post get；

关键代码：

    // get json data from network
    fetchData(){
        fetch(REQ_URL)
            .then((response) => response.json())
            .then((responseData) => {
                this.setState({
                    movies: responseData.movies
                });
            })
            .done();
    }


[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/project2-WebAsync/WebAsyncPage.js)



> [官网API地址](http://reactnative.cn/docs/0.26/textinput.html#content)
