---
layout: post
title: ReactNative
date: 2016-05-24
excerpt: "ReactNative 学习笔记 初篇 环境搭建和知识储备"
tags: [ReactNative]
comments: true
---


## ReactNative 第1节 必备的学习网站

### 官网
- [ReactNative 官网](https://facebook.github.io/react-native/) 
- [ReactNative CN](http://reactnative.cn/docs/0.26/getting-started.html)
- [React 官网](http://facebook.github.io/react/index.html)
- [React 中文](http://react-china.org/)

### 学习指南
- [React Native 学习指南](https://github.com/reactnativecn/react-native-guide)
- [React Native CN Github](https://github.com/reactnativecn)
- [awesome-react-native](https://github.com/jondot/awesome-react-native)

### blog & 视频
- [江清清的技术专栏](http://www.lcode.org/) 
- [React Native布局实践 开发京东客户端首页]( http://blog.csdn.net/yuanguozhengjust/article/category/6058018)
- [手把手教ReactNative](http://reactnative.cn/post/759)
- [推荐5个值得学习React Native的开源项目](http://www.tuicool.com/articles/BrIvMvE)

### JS lib
- [JS库查询](https://cdnjs.com/)
- [JS COACH](https://js.coach/react-native?search=viewp&page=2)


##  ReactNative 第2节  环境搭建 ##
[ReactNative环境搭建](https://yunpan.cn/cREqTK2G6zGmr)
（提取码：751f）

## ReactNative 第3节 IDE

1.Facebook推荐IDE——基于Atom的[nuclide](https://nuclide.io/)
  
  nuclide是Facebook在开源React Native框架时同时开源的IDE，是基于Github发布的开源编辑器Atom开发的React开发插件
  
  由于有Facebook的支持，对React.js和React Native的支持比较好。但一个很严重的问题是，Atom基于浏览器，性能是很严重的问题。
  
  免费，但是仅支持Mac: is not currently supported on Windows.

2.永恒的经典——Sublime 

  做JS的FE必然都知道Sublime。需要安装一些插件来支持React Native

3.全能型选手——WebStorm 11

  跟Android Studio很类似的布局。熟悉Android Studio上手WebStorm比较快，使用很顺手。

  WebStorm自带的插件也比较好用，提示非常智能，不用安装什么插件就能上手开发了
  
   收费，有破解版

  [在webstom里面配置运行RN的方案](http://reactnative.cn/post/539) 
  
<font color='red' size=4>4.微软推荐的Visual Studio Code</font>
  
  [Visual Studio Code](https://code.visualstudio.com/b?utm_expid=101350005-21.ckupCbvGQMiML5eJsxWmxw.1) 是微软在2015年4月29号 Build 2015 大会上发布的免费跨平台代码编辑器 

  支持 Win/Mac/Linux 多平台开发。Visual Studio Code支持语法高亮、智能代码补全、自定义热键、括号匹配、代码片段、代码对比 Diff、GIT 等特性，并针对网页开发和云端应用开发做了优化。软件跨平台支持 Win、Mac 以及 Linux，运行流畅
  https://code.visualstudio.com/docs/c?utm_expid=101350005-20.jAsCkEFcTeqvtdr0STCN9g.2&start=true

  免费
  
  推荐的[插件](https://marketplace.visualstudio.com/VSCode)：
  
  https://marketplace.visualstudio.com/items?itemName=vsmobile.vscode-react-native
  
  https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint
  
  https://marketplace.visualstudio.com/items?itemName=alechp.react-toolbox-snippets


## ReactNative 第4节 建议学习的相关技术

1. HTML+CSS3
   
   有Android和WPF开发经验的程序猿，可能更容易将XML和XAML的思想引入JSX中。
   
   而且React Native的布局样式，类似CSS3，有很多从CSS3中发展而来的字段和属性，所以建议学习一下CSS3。
   
2.React.js

  React Native是从React.js发展而来的。只是一个是面向Web，一个面向客户端。学习React.js有助于了解这种开发思想的核心，尤其是组件生命周期，
  
  生命周期和事件处理、数据绑定等功能几乎一模一样，参考React.js的原理和流程，非常有助于理解React Native。
  
3.Node.js、ES6
  
  React Native的代码结构来源于Node.js，其中require、exports的思想也是非常经典的。
  
  React Native用了很多基于ES6的思想来构建API，比如fetch，所以，学习一下新发布的ES6，不光是React Native，对其他JS开发也是必须的。
  
4.Android、iOS的JavaScript Bridge技术
 
  React Native内部实现了一套自己的JS通信技术，能够提供高效的通信，从而实现APP的动态改变、高效展示。所以有必要了解React Native的底层通信技术，以便后期对APP进行优化


##  ReactNative 第5节 测试ReactNativeProject ##

 1. cmd运行命令行：react-native init AwesomeProject
 2. cd AwesomeProject，cmd运行 react-native start
 3. cmd打开另外一个命令行。cd AwesomeProject，cmd运行react-native run-android

**如果是Device运行时，出现红色的一些提示。可以试试下面的操作**
（Android 5.0的系统）
1. cmd开一个新的命令行，运行 adb reverse tcp:8081 tcp:8081
2. 设备端Reload JS



## Tips ##

### node_modules文件名或扩展名太长如何删除 ##
步骤一：npm install -g rimraf

步骤二：rimraf node_modules

### Run on a Device 
adb reverse tcp:8081 tcp:8081

### 避免中文乱码
修改WebStorm默认保存的文件编码格式：File > Settings > Editor > File Encodings  

IDE Encoding 和 Project Encoding 改成 UTF-8




> [ReactNatvie IDE及其他相关基础技术](http://blog.csdn.net/yuanguozhengjust/article/details/50468561)