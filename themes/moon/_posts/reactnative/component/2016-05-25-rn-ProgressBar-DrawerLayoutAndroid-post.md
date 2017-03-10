---
layout: post
title: ReactNative 学习笔记 Component - ProgressBar，DrawerLayoutAndroid
date: 2016-06-07
excerpt: "ProgressBar，DrawerLayoutAndroid"
tags: [ReactNative]
comments: true
---

## Component - ProgressBar

封装了Android平台上的ProgressBar的React组件。这个组件可以用来表示应用正在加载或者有些事情正在进行中。

[ProgressBarAnroid](http://reactnative.cn/docs/0.26/progressbarandroid.html#content)

关键代码：
            <View style={[styles.flex,{marginTop:45}]}>
                <ProgressBarAndroid styleAttr="LargeInverse" />
           </View>
    

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/ProgressBarLesson.js)

> [官网API地址](http://reactnative.cn/docs/0.26/textinput.html#content)



## Component - DrawerLayoutAndroid


封装了平台DrawerLayout（仅限安卓平台）的React组件。抽屉（通常用于导航切换）是通过renderNavigationView方法渲染

并且DrawerLayoutAndroid的直接子视图会成为主视图（用于放置你的内容）。

导航视图一开始在屏幕上并不可见，不过可以从drawerPosition指定的窗口侧面拖拽出来

抽屉的宽度可以使用drawerWidth属性来指定。


关键代码：

    render() {
        var navigationView = (
            <View style={[styles.flex]}>
                <Text style = {styles.title}>This is Draw</Text>
            </View>
        );
        return (
            <DrawerLayoutAndroid
                drawerWidth={100}
                drawerPosition={DrawerLayoutAndroid.positions.Right}
                renderNavigationView={() => navigationView} >
                <View style={styles.drawview}>
                    <Text style={styles.drawitem}>Hello</Text>
                    <Text style={styles.drawitem}>Draw is Right</Text>
                </View>
            </DrawerLayoutAndroid>
        );
    }



[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/DrawerLayoutLesson.js)


> [ReactNative API 文档](http://reactnative.cn/docs/0.26/getting-started.html)
