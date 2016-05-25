---
layout: post
title: ReactNative
date: 2015-05-25
excerpt: "ReactNative 学习笔记 （一）"
tags: [react, technology]
comments: false
---


## 第一节 必备的官网和blog ##
- FaceBook ReactNative官网 ：[https://facebook.github.io/react-native/](https://facebook.github.io/react-native/ "https://facebook.github.io/react-native/") 
- ReactNative中文blog ： [http://reactnative.cn/blog.html](http://reactnative.cn/blog.html "http://reactnative.cn/blog.html")
- React 官网：[http://facebook.github.io/react/index.html](http://facebook.github.io/react/index.html "http://facebook.github.io/react/index.html")
- React 中文：[http://react-china.org/](http://react-china.org/ "http://react-china.org/")
- 手把手教ReactNative：[http://reactnative.cn/post/759](http://reactnative.cn/post/759 "http://reactnative.cn/post/759")
- JS库查询 [https://cdnjs.com/](https://cdnjs.com/ "https://cdnjs.com/")

## 第二节  环境搭建 ##
[https://yunpan.cn/cPnMZrdLHiVu4 ](https://yunpan.cn/cPnMZrdLHiVu4  "ReactNative环境搭建")
（提取码：dc13）


## 第三节 测试ReactNativeProject ##

 1. cmd运行命令行：react-native init AwesomeProject
 2. cd AwesomeProject，cmd运行 react-native start
 3. cmd打开另外一个命令行。cd AwesomeProject，cmd运行react-native run-android

**如果是Device运行时，出现红色的一些提示。可以试试下面的操作**
（Android 5.0的系统）
1. cmd开一个新的命令行，运行 adb reverse tcp:8081 tcp:8081
2. 设备端Reload JS


## 第四节 ReactNative 实例 ## 

### ReactNative 实例效果 ###
http://i.imgur.com/KVtcb1c.jpg

## 源码 ##
ReactNative中有很多跟React的语法不同。例如：

### 样式 ###
.label -> lable:

pading:0 3px 3px 0; -> padingTop:0, ...... paddingLeft 0，

### 元素 ###
<span>margin</span> -> <View>margin</View>

### 书写格式 ###
justity-content: -> justifyContent:

### 源码下载 ###


## React Native  第五节 实战之JSX入门
React是由ReactJS与React Native组成。其中ReactJS是Facebook开源的一个前端框架。
React Native是ReactJS思想在native上的体现！
JSX并不是一门新的语言，仅仅是个语法糖，允许开发者在JavaScript中书写HTML语法。最后每个HTML标签都转化为JavaScript代码来运行

### 1. 环境 ###
依赖于react JS环境
在Code中可以看到下面
`<script type="text/javascript" src="../react/react.js"></script> 核心文件`
`<script type="text/javascript" src="../react/react-dom.js"></script> 核心文件`
`<script type="text/javascript" src="../react/browser.min.js"></script> JSX解码器`

### 2. 载入方式 ###
inner or outer load.
inner like this:
`return <h1> Hello{ this.props.name },ASUS HZ! </h1>;`

### 3. 标签 ###
HTML标签，例如`<h3>` </br>
ReactJS标签（自定义组件标签)。
<font color=red>首字母一定要大写</font></br>
例如HelloMessage标签:
`ReactDOM.render( <HelloMessage name = "React" / >）`

### 4. 转换 解析器 ###
browser.min.js会负责把HTML标签转换成JS code在浏览器运行
例如 `<h3>in</h3>`  转换 React.createElement("h3",null,"in"); 返回ReactElement对象

### 5. 表达式 ###
执行JS表达式 <br/>
`var hello = "hello world!"`
`<h1>{hello}</h1>`
`解析过程：React.createElement("h1",null,hello); `

### 6. 注释 ###
单行 // <br/>
多行 /* */

### 7. 属性 ###
`var msg = <h1 width="10px">hello prop</h1>`  <br/>
`解析过程：React.createElement("h1",{width:}, "hello prop"); `

### 8. 延展属性 ###
使用ES6的语法
`var prop = {};`  <br/>
`prop.foo = "1"; `  <br/>
`prop.bar = "2"; `<br/>
`<h1 {...prop} foo="2">Hello</h1> (三个点是便利对象，并把值赋给h1` <br/>
`解析过程：React.createElement("h1",React.__spread({},prop,{foo:"2"}), "Hello"); `

### 9.自定义属性 ###
html5：以data-开头的自定义属性可渲染到页面<br/>
`return <h1 height="100" data-test="test" test="test"> Hello{ this.props.name },ASUS HZ! </h1>; `<br/>
如果是这样写只有height(h5自带属性），data-test会是被识别。test属性会被忽略掉

### 10. 样式 ###
需要style属性定义.<br/>
`return <h1 style={{color: '#ffff00', fontSize:'30px'}}> Hello{ this.props.name }!</h1>;`<br/>
style里面有两个{}。外层{}按照JSX语法，内层{}是JavaScript对象

### 11. 事件绑定
用HTML标签和ReactJS标签实现button click.  <br/>
代码下载 https://yunpan.cn/cSdswBA7b53jW （提取码：5ce3）

> https://segmentfault.com/a/1190000002646155


##  ReactNative 第六节 实战之ReactJS 组件和生命周期
ReactNative关键是ReactJS思想在Native上的体现。 <br/>
ReactJS核心思想：组件化。  <br/>
每个组件维护自动的状态和UI，状态变化时，自动重新渲染。多个组件组成了ReactJS应用。
### 组件
看Sample文件 https://yunpan.cn/cSLnXwpc4u3tI （提取码：6760）<br/>
下面几个常用项：

- React是全局对象，顶层API与组件API
- React.createClass 创建组件的方法
- ReactDOM.render 渲染，将制定组件渲染到指定DOM节点
- render: function () 组件级API

### 组件生命周期（Sample文件）

1. 创建
getDefaultProps：处理props的默认值 在React.createClass调用<br/>

        // 1. create-------------------------------------------------------------------------

        getDefaultProps: function () {// 在创建类的时候被调用。this.props该组件的默认属性
            console.log("getDefaultProps");
            return {};
        },
2. 实例化
React.render(<Msg 启动之后开始实例化<br/>
getInitialState、componentWillMount、render、componentDidMount<br/>
state：组件的属性，主要是用来存储组件自身需要的数据，每次数据的更新都是通过修改state属性的值。
ReactJS内部会监听state属性的变化，有变化就会主动触发组件的render方法来更新虚拟DOM结构<br/>
虚拟DOM：将真实的DOM结构映射成一个JSON数据结构<br/>

        // 2. 实例化------------------------------------------------------------------------

        getInitialState: function () {// 初始化组件的state值，其返回值会赋值给组件的this.state属性。
        // 获取this.state的默认值
            console.log("getInitialState");
            return {};
        },

        componentWillMount: function () {//在render之前调用此方法。业务逻辑的处理都应该放在这里，如对state的操作等
            console.log("componentWillMount");
        },

        render: function () {
            console.log("render");
            return <h1> Hello {this.props.name}! </h1>;
        },

        componentDidMount: function () { //render方法之后调用.ReactJS会使用render方法返回的虚拟DOM对象来创建真实的DOM结构
            //组件内部可以通过this.getDOMNode()来获取当前组件的节点
            console.log("componentDidMount");
        },
3. 更新<br/>
主要发生在用户操作之后或父组件有更新的时候，此时会根据用户的操作行为进行相应的页面结构的调整
componentWillReceiveProps、shouldComponentUpdate、componentWillUpdate、render(diff 算法）、componentDidUpdate

        //3. 更新阶段-------------------------------------------------------------------------------------------
        //主要发生在用户操作之后或父组件有更新的时候，此时会根据用户的操作行为进行相应的页面结构的调整

        componentWillReceiveProps: function () {//该方法发生在this.props被修改或父组件调用setProps()方法之后
            //调用this.setState方法来完成对state的修改
            console.log("componentWillRecieveProps");
        },
        shouldComponentUpdate: function () { //用来拦截新的props或state，根据逻辑来判断
            //是否需要更新
            console.log("shouldComponentUpdate");
            return true;
        },
        componentWillUpdate: function () {//shouldComponentUpdate返回true的时候执行
            //组件将更新
            console.log("componentWillUpdate");
        },
        componentDidUpdate: function () {//组件更新完毕，我们常在这里做一些DOM操作
            console.log("componentDidUpdate");
        },

4. 销毁<br/>
销毁时被调用，通常做一些取消事件绑定、移除虚拟DOM中对应的组件数据结构、销毁一些无效的定
时器等工作

        //4. 销毁阶段------------------------------------------------------------------------------------------
        componentWillUnmount: function () { //销毁时被调用，通常做一些取消事件绑定、移除虚拟DOM中对应的组件数据结构、销毁一些无效的定时器等工作
            console.log("componentWillUnmount");
        },

##  ReactNative 第七节 实战之ReactJS 组件通讯
React的核心思想是组件化，必然需要知晓组件的通讯</br>
ReactJS组件关系是一层套一层，DOM结构。就像HTML的标签。组织结构比较清晰。</br>
组件分为：父组件，子组件</br>
1.子组件如何调用父组件的方案
 
    方法：this.props
2.父组件如何调用子组件
    
    方法：ReactDOM.findDOMNode(this.refs.child)
    首先给child组件去个名字：ref="child"；然后用this.refs去获取
               
[Sample请点击这里](https://github.com/vivianking6855/reactdemo/blob/master/lesson/lesson6_community.html)



## Awesome React Native

[Awesome React Native ](https://github.com/jondot/awesome-react-native)

[推荐5个值得学习React Native的开源项目](http://www.tuicool.com/articles/BrIvMvE)

[React 入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html)


## Run on a Device ##
adb reverse tcp:8081 tcp:8081

> [http://reactnative.cn/post/759](http://reactnative.cn/post/759)


## Tips ##

### node_modules文件名或扩展名太长如何删除 ##
步骤一：npm install -g rimraf
步骤二：rimraf node_modules