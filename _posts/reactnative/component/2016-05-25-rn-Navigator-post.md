---
layout: post
title: ReactNative 学习笔记 Component - Navigator
date: 2016-05-27
excerpt: "Navigator"
categories: ReactNative
tags: [ReactNative]
comments: true
---

## Component - Navigator

应用一般是由很多页面组成。多页面的切换需要由路由或导航。

RN中对应的控件是：Navigator（Android/IOS)，NavigatorIOS


- Navigator，NavigatorIOS都可以用来管理应用中的Page导航。导航器建立了一个路由栈，用来弹出，推入或者替换路由状态。

- 他们和H5的history API类似。

- NavigatorIOS用了IOS中的UINavigationController类。只能用于IOS。

- Navigator则完全用js重写了一个类似功能的React组件。因此Navigator可以兼容Android和IOS。

- NavigatorIOS 轻量级，封住程度高，受限的API设置。较Navigator不太方便。是三方团队提供。非官方提供。


使用导航器可以让你在应用的不同场景（页面）间进行切换。导航器通过路由对象来分辨不同的场景。

利用renderScene方法，导航栏可以根据指定的路由来渲染场景。

可以通过configureScene属性获取指定路由对象的配置信息，从而改变场景的动画或者手势。

查看Navigator.SceneConfigs来获取默认的动画和更多的场景配置选项。


实例逻辑解析：

1. Navigator默认页面是List
2. List点击每一项，进入Detail页面。传递title和getUser方法给Detail页面
3. Detail页面通过title获取user信息，然后调用getUser方法设定List的user state
4. 点击Detail返回后，List页面显示user信息


关键代码：
    
    List 的_pressButton

     _pressButton(title) {
        const { navigator } = this.props;
        this.state.title = title;
        //为什么这里可以取得 props.navigator?请看上文:
        //<Component {...route.params} navigator={navigator} />
        //这里传递了navigator作为props
        const  self = this;
        if(navigator) {
            navigator.push({
                name: 'Detail',
                component: Detail,
                params: {
                    title: this.state.title,
                    getUser:function(user){
                        self.setState({
                            user: user
                        })
                    }
                }
            })
        }
    }
    
    Detail的_pressButton
    _pressButton() {
        const { navigator } = this.props;
        if(this.props.getUser){
            let user = USER_MODELS[this.props.title];
            this.props.getUser(user);
        }
        if(navigator) {
            //把当前的页面pop掉，返回到了上一个页面:List
            navigator.pop();
        }
    }
 
                
[Sample Code](https://github.com/vivianking6855/ReactNativeProject/blob/rncomponent/TwoReactNative/app/NavigatorLesson.js)


> [详细的教程](http://bbs.reactnative.cn/topic/20/新手理解navigator的教程)
> [官网API地址](http://reactnative.cn/docs/0.26/navigator.html#content)