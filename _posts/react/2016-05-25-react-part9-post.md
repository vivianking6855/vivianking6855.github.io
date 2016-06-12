---
layout: post
title: ReactNative
date: 2015-06-03
excerpt: "ReactNative 学习笔记 9 Component - Image，Picker"
tags: [react]
comments: true
---

## Component - Image

RN Image组件调用途径途径有很多：网络图片、本地图片、照相机图片

主要属性：

- resizeMode enum('cover', 'contain', 'stretch') 决定当组件尺寸和图片尺寸不成比例的时候如何调整图片的大小。

    - cover: 在保持图片宽高比的前提下缩放图片，直到宽度和高度都大于等于容器视图的尺寸（如果容器有padding内衬的话，则相应减去）。译注：这样图片完全覆盖甚至超出容器，容器中不留任何空白。

    - contain: 在保持图片宽高比的前提下缩放图片，直到宽度和高度都小于等于容器视图的尺寸（如果容器有padding内衬的话，则相应减去）。译注：这样图片完全被包裹在容器中，容器中可能留有空白

    - stretch: 拉伸图片且不维持宽高比，直到宽高都刚好填满容器。

- source {uri: string}, number 

    - uri是一个表示图片的资源标识的字符串，它可以是一个http地址或是一个本地文件路径（使用require(相对路径)来引用）。


实例逻辑解析：

1. 输入变动时列出提示内容
2. 添加state show 和 value来标记提示显示状态和input的内容

关键代码：

                    <Image style={styles.img}
                         resizeMode="contain"
                         source={{uri:this.state.imgs[this.state.count]}}/> //  网络图片
                        //source={require('./xiaoman2016_24.png')}   本地图片
    

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/ImageLesson.js)

> [官网API地址](http://reactnative.cn/docs/0.26/textinput.html#content)



## Component - Picker

类似Andorid的Spinner

关键代码：

    render() {
        return (
            <View style={[styles.flex,{marginTop:45}]}>
                <Text> Picker组件 </Text>
                <Picker selectedValue={this.state.language}
                    onValueChange={lang => this.setState({language:lang})}
                    mode="dialog"
                     style={{color:'#F00'}} >
                    <Picker.Item label="111" value="111" />
                    <Picker.Item label="222" value="222" />
                </Picker>
                <Text> {this.state.language} </Text>
           </View>
        );
    }

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/PickerLesson.js)


> [ReactNative API 文档](http://reactnative.cn/docs/0.26/getting-started.html)
