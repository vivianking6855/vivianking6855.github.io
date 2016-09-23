---
layout: post
title: ReactNative 学习笔记 Community- 组件，页面通讯
date: 2016-06-30
excerpt: "ReactNative 学习笔记 Community- 组件，页面通讯"
tags: [ReactNative,学习笔记]
comments: true
---

## 组件，页面通讯

### 1.父子组件

ReactJS中数据的流动是单向的

父组件传递数据给子组件: 可以通过设置子组件的props传递数据给子组件。

子组件改变父组件的数据: 可以在父组件中传一个callback(回调函数)给子组件，子组件调用callback可改变父组件的数据。

实例：点击子组件，改变父组件的背景色
 
    
    export default class SampleOne extends Component {
        constructor(props) {
            super(props);
            this.state = {
                color: 'white',
            };
        }
    
        // 改变curItem的回调函数 
        _changeColor(color) {
            this.setState({ color: color });
        }
    
        render() {
            return (
                <View style={{ backgroundColor: this.state.color, flex: 1 }}>
                    <Child text='Child one' color='red'
                        changeColor={ (color) => this._changeColor.bind(this, color) }/>
                    <Child  text='Child two' color='white'
                        changeColor={ (color) => this._changeColor.bind(this, color) }/>
                </View>
            );
        }
    }
    
    class Child extends Component {
        render() {
            return (
                <Text style = {{ fontSize: 50, margin: 20 }}
                    onPress={this.props.changeColor(this.props.color) }>
                    {this.props.text}
                </Text>
            );
        }
    }
    
   
### 2.兄弟组件

当两个组件不是父子关系，但有相同的父组件时，将这两个组件称为兄弟组件。

兄弟组件不能直接相互传送数据，此时可以将数据挂载在父组件中。

由两个组件共享：如果组件需要数据渲染，则由父组件通过props传递给该组件；

如果组件需要改变数据，则父组件传递一个改变数据的回调函数给该组件，并在对应事件中调用。

实例：点击子组件，改变兄弟组件的字体颜色


    export default class SampleTwo extends Component {
        constructor(props) {
            super(props);
            this.state = {
                child1color: 'blue',
            };
        }
    
        _changeItemColor(color) {
            this.setState({ child1color: color });
        }
    
        render() {
            return (
                <View style={{flex: 1 }}>
                    <Child ref='child1' text='Child one' color={this.state.child1color}/>
                    <Child  ref='child2' text='Child two' color='purple'
                        changeBrotherColor={ (color) => this._changeItemColor(color) }/>
                </View>
            );
        }
    }
    
    class Child extends Component {
    
        _changeBrotherColor() {
            this.props.changeBrotherColor && this.props.changeBrotherColor('green');
        }
    
        render() {
            return (
                <Text style = {{ fontSize: 60, margin: 20, color: this.props.color }}
                    onPress={this._changeBrotherColor.bind(this)}>
                    {this.props.text}
                </Text>
            );
        }
    }


### 3.全局事件


1和2中的方法在组织结构非常深的时候，将会是一个噩梦。我们可以用全局事件来避免这些麻烦

使用事件来实现组件间的沟通：改变数据的组件发起一个事件，使用数据的组件监听这个事件，在事件处理函数中触发setState来改变视图或者做其他的操作。

使用事件实现组件间沟通脱离了单向数据流机制，不用将数据或者回调函数一层一层地传给子组件。

事件模块可以使用如EventEmitter或PostalJS这些第三方库，也可以自己简单实现一个。

使用RCTDeviceEventEmitter sample code:


    import RCTDeviceEventEmitter from 'RCTDeviceEventEmitter';
    
    export default class SampleThree extends Component {
        render() {
            return (
                <View style={styles.container}>
                    <ChildInput />
                    <ChildShow />
                </View>
            );
        }
    }
    
    class ChildInput extends Component {
    
        handleUpdateChange(text) {
            RCTDeviceEventEmitter.emit('change', text);
        }
    
        render() {
            return (
                <View style={{ justifyContent: 'center', alignItems: 'center', backgroundColor: '#F5FCFF' }}>
                    <TextInput onChangeText={(text) => this.handleUpdateChange(text) } style={{ width: 200, height: 40, borderColor: 'gray', borderWidth: 1 }} />
                </View>
            );
        }
    }
    
    class ChildShow extends Component {
        constructor(props) {
            super(props);
            this.state = {
                text: '',
            };
        }
    
        componentDidMount() {
            let me = this;
            RCTDeviceEventEmitter.addListener('change', function (text) {
                me.setState({
                    text: text
                })
            })
        }
    
        render() {
            return (
                <View style={{ justifyContent: 'center', alignItems: 'center', backgroundColor: '#F5FCFF' }}>
                    <Text>{this.state.text}</Text>
                </View>
            );
        }
    }
    
    const styles = StyleSheet.create({
      container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
      },
    });



但是正因为全局事件脱离了单向数据流机制，从而使得数据的流向变得的不可预期。


### 4. Navigator didfocus 和  Navigator传递方法通讯

#### 监听didfocus事件（不推荐）
用在有Navigator的scenario.

在需要更新数据的组件didmount时重新load storage数据加载

关键code


    componentWillMount() {
        console.log('componentWillMount');
        let me = this;
        let willcallback = (event) => {
            console.log('will enter ' + this.props.name);
        };

        let didcallback = (event) => {
            console.log(
                'didfocus event ',
                {
                    route: event.data.route,
                    target: event.target,
                    type: event.type
                });

            // homeview did mount
            if ('HomeView' === event.data.route.name && 'didfocus' === event.type) {
                console.log('didfocus HomeView');
                this.updatetext && me.setState({ text: this.updatetext });
            }
        };

        // observe focus change events from this component
        this._listeners = [
            // 该会页面每次进行切换之前调用
            this._navigator.navigationContext.addListener('willfocus', willcallback),

            // 在每次页面切换完成或者初始化之后进行调用该方法。该参数为新页面的路由
            this._navigator.navigationContext.addListener('didfocus', didcallback),
        ];
    }

### Navigator传递方法

用在有Navigator的scenario


    in SampleFive.js

    _navigateToSub() {
        console.log('_navigateToSub');
        const self = this;
        this._navigator && this._navigator.push({
            name: 'SampleFiveSub',
            component: SampleFiveSub,
            params: {
                updateText: (text) => {
                    self.updatetext = text;
                }
            }
        });
    }
    
    in SampleFileSub.js
    
    _back() {
        console.log('SampleFiveSub _back');
        this.props.updateText && this.props.updateText(this.text)
        const {navigator} = this.props;
        navigator && navigator.pop();
    }


### 4. Redux

Redux可以查看章节 [Redux](http://vivianking6855.github.io/redux/)


## 小结

 - 简单的的组件沟通可以用传props和callback的方法实现。随着项目规模的扩大，组件就会嵌套得越来越深，这时候使用这个方法就有点不太适合。

 - 全局事件可以让组件直接沟通，但频繁使用事件会让数据流动变得很乱。

 - 简单的处理逻辑也可以用Navigator传递callback方法

 - 使用redux可以让你整个项目的数据流向十分清晰，但是很容易会出现组件嵌套太深的情况，events和context都可以解决这个问题。

 - Transdux是一个类redux框架，使用这个框架可以写出比redux简洁的代码，又可以得到redux的好处。


> [ReactJS组件间沟通的一些方法](http://my.oschina.net/fzxgg/blog/607421?fromerr=VNsQSR6N)
