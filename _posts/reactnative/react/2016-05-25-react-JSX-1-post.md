---
layout: post
title: React 实战之JSX - 入门
date: 2016-05-25
excerpt: "React 实战之JSX - 入门"
categories: ReactNative
tags: [React]
comments: true
---

##  第1节 实例

### React 实例效果 ###
http://i.imgur.com/KVtcb1c.jpg

## React语法 ##
ReactNative中有很多跟React的语法不同。例如：

### 样式 ###
.label -> lable:

pading:0 3px 3px 0; -> padingTop:0, ...... paddingLeft 0，

### 元素 ###
<span>margin</span> -> <View>margin</View>

### 书写格式 ###
justity-content: -> justifyContent:

### 源码下载 ###


## 第2节 实战之JSX入门
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
