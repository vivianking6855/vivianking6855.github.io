---
layout: post
title: Android架构总结
date: 2018-5-30
excerpt: "Android架构总结"
categories: Android
tags: [Android架构]
comments: true
---


# 简介

为什么需要架构？

当项目非常庞大时，项目的可维护性，可扩展性，易测性显得尤其的重要。

好的架构能够帮助我们更好实现：

•易维护
•易测试
•高内聚
•低耦合

Bob大神的Architecture is About Intent, not Frameworks. 个人理解是：架构应该是面向意图，而不是面向片段。

因此在架构的分层最好依据不同的意图来划分。

另外业务逻辑庞大的项目后期，必须对项目进行模块化的拆分。可以通过组件化，插件化等技术来实现。

这里我们仅针对app的架构讨论，不会涉及深入的组件化和插件化内容。

# 我们的CleanArchitecture架构

[CleanArchitecture架构](https://github.com/vivianking6855/android-architecture/tree/master/CleanArchitecture)是在项目实践中不断总结出的Clean Architecture

是我们产品研发时尽可能遵循的原则，我们的期望是任何一个内部环节对外部是解耦的，没有依赖关系。

这也是架构设计的核心：独立性和可测试性；二者是相符想成的。

比如纯Java的可以用JUnit等测试，Android业务相可以用Android Instrumentation等测试。

效果图：

![](https://i.imgur.com/OAJV0s7.jpg)

我们的项目基本包含四个层次

![](https://i.imgur.com/5X0VIG0.jpg)

其中数据层，业务无关层分别是独立的library；展示层和业务公有层在app module中； 展示层和数据层的通过接口方式现实数据监听

1. 展示层business

	业务展示层采用MVP架构
    
	- V: activity, fragment, view
    - M: model
    - P: presenter处理数据逻辑，使用data module中的各个repository
        - P和V的解耦合可以通过Lisenter，EventBus, RxAndroid/RxJava等方式
        - P调用使用data module中的各个repository，异步数据监听可以通过data层的接口来实现
	
	实际使用中展示层是依据功能模块分成各个独立的模块，例如Home, User等。
	
	每个模块都包含view,model,presenter,listerner,adapter，fragment 六个标准文件夹，用户也可以按照自己的需要再客制化

2. 业务公共层 businesscommon

    - router 页面跳转器
    - utils 业务公用工具
    - base 业务封装基类
   
3. 数据层 data module
    
	- repository 提供给展示层的数据接口（例如UserProvider等）
    - listener 提供给上层的数据变化监听接口
    - cache 缓存
    - db 数据库
    - entity 数据单元，其中可解析的entity多用于网络请求
    - exception 异常
    - net 网络相关
    - task 各类任务线程池

4. 业务无关库 common module
    - 业务逻辑无关的一些公用库
    - 也有使用到已发布的JCenter上的库

除此之外还有debug模块来管控log输出和调试

# MVP架构说明

- V:是一个接口
- P:BasePresenter持有V
- BaseMVPActivity持有P extends BasePresenter<V>，且implements V的接口

		--------BaseMVPActivity
	
		public abstract class BaseMVPActivity<V, P extends BasePresenter<V>> extends BaseActivity {
	
		--------应用实例
	
		1. 定义V接口

			public interface IHomeDisplayer {
			    void onDisplay(UserModel model);
			}

		2. HomePresenter继承BasePresenter<IHomeDisplayer>

			public class HomePresenter extends BasePresenter<IHomeDisplayer> {

			数据加载成功后调用V的方法（viewWeakRef.get()获取View的实例）
            viewWeakRef.get().onDisplay(DataTransaction.transform(user));
	
		3. HomeActivity继承BaseMVPActivity，实现V IHomeDisplayer的方法

			public class HomeActivity extends BaseMVPActivity<IHomeDisplayer, HomePresenter> implements IHomeDisplayer {
			
			    @Override
			    public void onDisplay(UserModel model) {
			        if (model != null) {
			            resultTV.setText(model.getUserId());
			        }
			    }

# 待优化

- 组件化部分尚未加入
- JUnit, espresso等测试用例模板尚未加入

后续项目复杂度提升，实践后再更新出来。

# 其他建议

- 尽可能搭配组件化/插件化：注意独立模块的单独调试和继承测试
- 动态加载（轻量级插件化）：若app对更新时效性有要求，需要。例如金融类app
- 性能和测试
    - 对app核心行为和各个组件交互和生命周期的监控，后期可以搭配bug服务器平台跟trace log一起上传，以便于bug的分析和追踪（LogMan写入文件)
    - 建议接入移动测试平台 

# [Vivian 的Android架构](http://vivianking6855.github.io/tag/#Android%E6%9E%B6%E6%9E%84-ref)     

# 开源架构

著名的[Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)

查看知乎[的一种更清晰的Android架构](https://zhuanlan.zhihu.com/p/20001838)

# Reference

[一种更清晰的Android架构](https://zhuanlan.zhihu.com/p/20001838)

[Android Architecture](https://github.com/android10/Android-CleanArchitecture)

[Uncle Bob: Architecture is About Intent, not Frameworks](https://www.infoq.com/news/2013/07/architecture_intent_frameworks)