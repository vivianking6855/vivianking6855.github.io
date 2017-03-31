---
layout: post
title: Android Disign And Refactoring 
date: 2017-3-30
excerpt: "Android Disign And Refactoring "
categories: Android
tags: [Android 进阶]
comments: true
---

# 前言

对于一个App来讲，重构是必要。除非一次性把它设计好。

# 什么时候做重构

一般来说，产品的需求是不断增加和变动的。 App的是否需要重构可以从下面几个角度来综合判断。

1. 如果不重构，会导致很严重的功能问题。
    - 必要时可能需要砍掉一些功能
    - 一定要跟planner一起Sync
2. 是否有空余的开发人员来做重构。
    - 可以创建新的分支开单独存放重构code
    - 搭配Q测试验证Pass
    - 分阶段发布（10%, 50%, 100%）
3. 重构计划要事先提出。一次重构最好不超过2W。
    - 如要要重构打的模块或者底层架构，可以考虑分阶段重构。把重构分为几次迭代
    - 分期上线，先重构解决最严重的问题

# 重构

重构可以分为下面几个部分

- 重构规划项目结构
- 为Activity定义新的生命周期
- 统一事件编程模型
- 实体化编程
- Adapter模板
- 类型安全转换函数

# 1. 重构规划项目结构

## 主项目

主项目这里列出了两种思路的项目结构

### 1）全部按照模块来划分

- 各个独立业务模块：每个模块中都包含自己的activity, adapter, entity，engine（例如splash，UserInfo等）
- db： sqllite相关逻辑封装
- ui: 所有自定义控件
- utils： 所有公用方法
- Interfaces： 真正意义上的接口，命名以I作为开头
- listener： 基于Listener的接口，命名以On作为开头

### 2）Activity按照模块划分

- activity: 按照模块划分，把不同Activity放到不同的模块包下面
- engine： 所有业务相关的类
- adapter: 所有的适配器
- entity: 所有的实体
- db： sqllite相关逻辑封装
- ui: 所有自定义控件
- utils： 所有公用方法
- Interfaces： 真正意义上的接口，命名以I作为开头
- listener： 基于Listener的接口，命名以On作为开头

这里Activity按照模块拆分了，adapter和entity没有拆分。这是因为：

- 我们看Code的时候通常是从Activity看起。
- 此外Adapter逻辑大同小异。Entity中应该只有属性，如果确实需要，请把它已到engine包里面（当Entity有上百个时，我们可以考虑按照模块拆分）
- 每个Activity都有着很复杂的业务逻辑，所以Activity才是最重要的。

### 类库

不管是 1）还是2）都建议建立一个类库AndroidLib，将业务无关的逻辑转移到这里。

AndroidLib可以划分为几个部分：（全部都是业务无关）

- activity: Activity基类
- net: 网络底层封装
- cache: 缓存数据和图片
- ui: 自定义控件
- utils： 公用方法

### 小结

- 重构规划项目结构有的好处：
    - 每个文件只有一个单独的类，不要有嵌套类
    - 将Activity按照模块拆分后，可以迅速定位具体的页面
    - 类库方便重用
- 不建议在Activity中嵌套adapter,entity. 
- 个人比较喜欢项目结构1），因为移除某个功能非常的便捷。
- 这里仅列出两个示例的项目结构，大家可以依照自己的需求定制合适的项目结构。

# Reference

> 《App研发录》

> [App研发录作者 Blog](http://www.cnblogs.com/jax/p/4656789.html)