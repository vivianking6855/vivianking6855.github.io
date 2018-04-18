---
layout: post
title: 优化规范 - Size
date: 2018-4-18
excerpt: "优化规范 - Size"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}




# size优化规范

本文主要针对Android移动应用的size优化规范

## 1. 衡量标准

App的size很难有统一的标准，因为需求差异对size的影响非常大。

简单的app可能仅有2M，3M，而大型的app可能有20M，30M不等（例如：Wechat：58.5M，爱奇艺：29.94，支付宝57.99。应用宝数据）

虽然没有统一的标准，但是通过竞品对比，三方库依赖等多个方面来大约估算size标准

例如Contact app 评估如下

![](https://i.imgur.com/qzyWEy3.png)

如果是特殊功能确实需要额外的size，需要列出size增加的必要理由

## 2. 分析

分析步骤如下：

A.	查看app size分布


![](https://i.imgur.com/wQ17MXF.png)

B.	分析各项占比较高的可优化性

例如：res, assets 资源；lib库；dex 

## 3. 分析

size优化步骤如下：

A.	无用资源

- 使用AS工具：“Remove Unused Resources” 移除Unused资源 （需要注意反射调用的资源不能移除）
- Pad，多国语言等资源是否必要
    
B.	Res资源

- icon建议使用svg格式适配所有型号的专案 
- 无需透明度图片建议使用jpg而非png图片 

C.	Lib库

- lib库是否必要，能否仅加载arm so
- 使用针对智能手机优化过的库 

D.	Dex

- 移除无用代码，移除enum等

E.	Gradle编译

启用ProGuard 压缩

    buildTypes {
      release {
          minifyEnabled true //启用代码压缩,混淆
          shrinkResources true //启用资源压缩
	      zipAlignEnabled true
	  }
     }

zipAlignEnabled是对应用程序中的资源作对齐操作

使得在运行时Android与应用程序间的交互更加有效率。

这种方式能够让应用程序和整个系统运行得更快

# Reference

[性能优化-目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)