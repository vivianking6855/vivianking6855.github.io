---
layout: post
title: ReactNative
date: 2016-06-30
excerpt: "ReactNative 学习笔记 物理back键"
tags: [ReactNative]
comments: true
---

## 物理back键处理

### 需求 ：实现：点击back， 连续点击退出app


### 关键Code

    _onBackPress = (event) => {
        let nav = this.refs.navigator;
        let routers = nav.getCurrentRoutes();
        if (routers.length > 1) {
            nav.pop();
            return true;
        } else {
            if (this.lastBackPressed && this.lastBackPressed + 2000 >= Date.now()) {
                //两秒内连续点back退出
                return false;
            }
            this.lastBackPressed = Date.now();
            ToastAndroid.show(Strings.backAgainToExit, ToastAndroid.SHORT);
            return true;
        }
    }

    componentWillMount() {
        if (Platform.OS === 'android') {
            BackAndroid.addEventListener('hardwareBackPress', this._onBackPress);
        }
    }

    componetWillUnmount() {
        if (Platform.OS === 'android') {
            BackAndroid.removeEventListener('hardwareBackPress', this._onBackPress);
        }
    }




[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/HomeProject.js)

> [手把手教React Native实战之物理back键详解](http://www.reactnative.vip/forum.php?mod=viewthread&tid=66&extra=page%3D1%26filter%3Dtypeid%26typeid%3D9)
> 