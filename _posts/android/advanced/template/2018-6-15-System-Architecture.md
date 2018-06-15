---
layout: post
title: 软件架构
date: 2018-6-15
excerpt: "软件架构"
categories: Web
tags: [Web]
comments: true
---

* content
{:toc}




# 前言

软件架构（Software Architecture）就是软件的基本结构。

MVX（X泛指C - Controller、 P - Presenter、 VM- View Model），具体是指MVP,MVC,MVVM。

这三种是我们现在经常看到或讨论的UI架构模式。仅仅是表现层的架构（Presentation Pattern），不适合作为系统框架。

文中所说的软件架构是指体系架构。

# 常见软件架构

五种常见软件架构（摘录自App架构师）

1. 分层架构（Layered Architecture）

	最通用最常见的架构，也叫N层架构模式（n-tier architecture pattern)
	
	一般4层结构：

	- 表现层（presentation 界面）
	- 业务层（business 业务逻辑）
	- 持久层（persistence 数据提供）
	- 数据库层（database 数据存储）

	特性

	- 优点：层与层之间隔离；独立测试；新增和变更维护方便
	- 缺点：用户请求必须经过每一层才能抵达最后层。不过可以在业务层和持久层之间增加服务层（Server），针对不同业务逻辑封装通用接口。

	![](https://i.imgur.com/n4TAdxS.png)

2. 事件驱动架构（Event-Driven Architecture)

    一种流行的分布式异步架构模式

	- 优点：基于事件通讯，高度解耦，易于扩展和部署
	- 缺点：开发和测试不方便

	[大型网站为什么要使用分布式服务](https://blog.csdn.net/bobozai86/article/details/79245137)

	![](https://i.imgur.com/oz7RYzW.png)

3. 微内核架构（Microkernel Architecture)

	又称为插件架构（plug-in architecture)

	常用于基于产品的应用程序：主要功能和业务逻辑都通过插件实现

	包含：核心系统（core system)和插件模块(plug-in component)两种组件。核心系统通常只包含系统运行的最小功能，插件之间相互独立。

	![](https://i.imgur.com/n4q8Zo0.png)	

4. 微服务架构（Microservices Architecture）

	每个组件都作为一个独立单元进行部署，单元间通过远程通信协议（比如REST,SOAP）联系。应用和组件之间高度解耦，部署更为简单。

	最通用、最流行的微服务架构有：RESTful API模式、RESTful Applicaiton模式和集中消息模式3种。

	![](https://i.imgur.com/kr0PJ3k.png)

5. 基于空间的架构（Space-Based Architecture）

	也成为云架构（Cloud Architecture)

	主要是解决规模和并发的问题，不存在中央数据库，使用可复制的内存数据单元，扩展极其方便。

	云架构分为：处理单元（Processing Unit）和虚拟中间件（Virtual Middleware)两部分

	![](https://i.imgur.com/nyEOJ3S.png)

# 架构实践小结（摘录自App架构师）

架构设计也如同设计模式，目的都是更好的为业务服务，更好的进行业务扩展。

架构是对客观不足的妥协，规范是对主观不足的妥协。对项目来说，还是具体问题具体分析，结合业务灵活使用和变化。

- 第一种，分层用的最早也最多，在App开发中非常适合

	例如Android系统架构也是一种类似的分层架构（当然应用架构中会加入统一的消息机制来协调处理等）

	几乎大部分基础的App框架设计都可以采取这种架构

- 第三种，插件化，核心系统/内核提供运行环境，插件之间解耦。类似涉及功能插件化的业务可以借鉴这种架构
- 第二种（消息驱动），第四种（微服务架构）有类似需要涉及分布式部署业务的可以借鉴这种架构。

	例如云测试项目，分布式机构，基于REST 分发Task，基于RPC通信，每个手机都是一个独立的单元。

# 一个中小型项目的架构设计：基于分层架构设计 （摘录自App架构师）

组件库抽离，业务模块独立，通过Bridge完成模块之间的基础通信，复杂业务交互用消息总线交互。

组件化后的App架构设计图

![](https://i.imgur.com/CstFlcK.png)

![](https://i.imgur.com/UOIkB78.jpg)


# Reference

《App架构师》

《分布式系统概念和设计》

[学习分布式系统需要怎样的知识](https://www.zhihu.com/question/23645117)
