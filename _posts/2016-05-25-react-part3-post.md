---
layout: post
title: ReactNative
date: 2015-05-25
excerpt: "ReactNative 学习笔记 Part 3"
tags: [react, technology]
comments: true
---

##  ReactNative 第6节 实战之ReactJS 组件和生命周期
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

##  ReactNative 第7节 实战之ReactJS 组件通讯
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
