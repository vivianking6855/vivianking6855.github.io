---
layout: post
title: Android Design-Pattern （二）
date: 2017-7-4
excerpt: "Android Disign Pattern （二）"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 结构型模式7种

## 6. 适配器模式

### 简介

基于现有类所提供的服务，向客户提供接口，以满足客户的期望。也可以理解为将一个类的接口转换成客户希望的另外一个接口。

适配器模式的用意是要改变源的接口，以便于目标接口相容。

但是过多的使用适配器，会让系统非常零乱，不易整体进行把握。

比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

### 核心Code 

### 已有示例

不同的数据提供者使用一个适配器来向一个相同的客户提供服务。ListView或GridView的Adapter。

## 7. 桥接模式

### 简介

将抽象部分与它的实现部分分离，使它们都可以独立地变化。

### 核心Code 

### 已有示例

Window和WindowManager之间的关系。

在FrameWork中Window和PhoneWindow构成窗口的抽象部分

其中Window类为该抽象部分的抽象接口，PhoneWindow为抽象部分具体的实现及扩展。

而WindowManager则为实现部分的基类，WindowManagerImpl则为实现部分具体的逻辑实现。

## 8. 装饰模式

### 简介

动态地给一个对象添加一些额外的职责。就扩展功能而言，它比生成子类方式更为灵活。

### 核心Code 

### 已有示例

Activity继承自ContextThemeWrapper，ContextThemeWrapper继承自ContextWrapper，ContextWrapper才是继承自Context。

ContextWrapper就是我们找的装饰者。

## 9. 组合模式

### 简介

将对象组合成树形结构以表示“部分-整体”的层次结构。

### 核心Code 

### 已有示例

View和ViewGroup的组合

## 10. 外观模式

### 简介

为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，统一编程接口。

### 核心Code 

### 已有示例

ContextImpl

## 11. 享元模式

### 简介

运用共享技术有效地支持大量细粒度的对象。

### 核心Code 

### 已有示例

Message.obtainMessage通过重用Message对象来避免大量的Message对象被频繁的创建和销毁。

## 12. 代理模式

### 简介

为其他对象提供一个代理以控制对这个对象的访问。

### 核心Code 

### 已有示例

所有的AIDL都一个代理模式的例子。

假设一个Activity A去绑定一个Service S，那么A调用S中的每一个方法其实都是通过系统的Binder机制的中转，然后调用S中的对应方法来做到的。

Binder机制就起到了代理的作用。

# 设计模式专题

- [设计模式专题一：创建型模式5种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-1/) 
- [设计模式专题二：结构型模式7种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-1/) 
- [设计模式专题二：行为型模式11种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-1/) 

Github Code: [https://github.com/vivianking6855/android-advanced/tree/master/DesignPattern](https://github.com/vivianking6855/android-advanced/tree/master/DesignPattern)

# Reference

[Android开发中常见的设计模式](http://www.cnblogs.com/android-blogs/p/5530239.html)

[Android设计模式之23种设计模式一览](http://blog.csdn.net/happy_horse/article/details/50908439)

[《Android深入透析》之常用设计模式经验谈](https://my.oschina.net/u/2249934/blog/343441)

《android之大话设计模式》

[设计模式中英文对照](https://wenku.baidu.com/view/a216bfa651e79b896802269a.html)

[Android 设计模式](http://blog.csdn.net/banketree/article/details/24985607)