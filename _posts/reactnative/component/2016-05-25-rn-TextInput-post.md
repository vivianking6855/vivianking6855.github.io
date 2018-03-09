---
layout: post
title: ReactNative 学习笔记 Component - TextInput，Touchable
date: 2016-06-03
excerpt: "TextInput，Touchable"
categories: ReactNative
tags: [ReactNative]
comments: true
---

# Component - TextInput

TextInput是一个允许用户在应用中通过键盘输入文本的基本组件。

本组件的属性提供了多种特性的配置，譬如自动完成、自动大小写、占位文字，以及多种不同的键盘类型（如纯数字键盘）等等。

最简单的用法就是丢一个TextInput到应用里，然后订阅它的onChangeText事件来读取用户的输入。

它还有一些其它的事件，譬如onSubmitEditing和onFocus。



实例逻辑解析：

1. 输入变动时列出提示内容
2. 添加state show 和 value来标记提示显示状态和input的内容

关键代码：

                   <TextInput
                        keyboardType="web-search"
                        placeholder="请输入关键词"
                        value={this.state.value}
                        onChangeText={this.getValue.bind(this)}/>
    
    
    

[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/TextInputLesson.js)

> [官网API地址](http://reactnative.cn/docs/0.26/textinput.html#content)


# Component - Touchable
RN没有Web开发那样给元素绑定click时间。RN的Text组件有onPress事件毁掉，View组件没有。

但是实际开发通常需要View被点击。因此RN提供了三个组件来赋予被点击的能力（去包裹其他组件）。

三个Touchable类型组件：

- TouchableHighlight 高亮

- TouchableOpacity 透明度

- TouchableWithoutFeedback 无变化


            <View style={styles.flex}>
                <TouchableHighlight onPress={this.show.bind(this,'TouchableHighlight')}
                underlayColor='#E1F6FF'>
                    <Text style={styles.item}>TouchableHighlight</Text>
                </TouchableHighlight>
                <TouchableOpacity onPress={this.show.bind(this,'TouchableOpacity')}>
                    <Text style={styles.item}>TouchableOpacity</Text>
                </TouchableOpacity>
                <TouchableWithoutFeedback onPress={this.show.bind(this,'TouchableWithoutFeedback')}>
                    <Text style={styles.item}>TouchableWithoutFeedback</Text>
                </TouchableWithoutFeedback>
            </View>

不建议使用：onLongPress、onPressIn、onPressOut


> [ReactNative API 文档](http://reactnative.cn/docs/0.26/getting-started.html)
