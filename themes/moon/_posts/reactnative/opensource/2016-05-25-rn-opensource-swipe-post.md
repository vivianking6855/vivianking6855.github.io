---
layout: post
title: ReactNative 学习笔记 Open Source - react-native-swiper
date: 2016-06-22
excerpt: "react-native-swiper"
tags: [ReactNative]
comments: true
---

## react-native-swiper



Swiper component for React Native.

1. Install react-native first

    $ npm i react-native-swiper --save 
    
2. rm react-native first

    $ npm rm react-native-swiper --save 


关键代码：

    import Swiper from 'react-native-swiper'

    render(){
        return (
            <Swiper style={styles.wrapper} showsButtons={true}>
                <View style={styles.slide1}>
                    <Text style={styles.text}>Hello Swiper</Text>
                </View>
                <View style={styles.slide2}>
                    <Text style={styles.text}>Beautiful</Text>
                </View>
                <View style={styles.slide3}>
                    <Text style={styles.text}>And simple</Text>
                </View>
            </Swiper>
        );
    }


[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/project3-Swipe/SwipePage.js)

[More Example](https://github.com/leecade/react-native-swiper/tree/master/examples/examples)



> [官网API地址](http://reactnative.cn/docs/0.26/textinput.html#content)

> [react-native-swiper github](https://github.com/leecade/react-native-swiper)
