---
layout: post
title: Android 测试
date: 2018-06-11
excerpt: "Android 测试"
categories: Android 进阶
tags: [Android 进阶]
comments: true
lefttrees: true
---



# 简介

Android的测试有很多框架，基本单元测试框架有

- Java单元测试框架：Junit、Mockito、Powermockito等
- Android：Robolectric、AndroidJUnitRunner、Espresso等

AS默认的是junit, AndroidJunitRunner, espresso（6/11/2018 11:16:54 AM ）

# 开源test demo

1. googlesampls 的android-testing

	googlesampls提供[android-testing](https://github.com/googlesamples/android-testing)来展示四大自动化测试框架的使用

2. robolectric

	一款不依赖于Android设备的单元测试框架，

	sample中列举了如何对Android四大组件和常见功能测试的用例，3.2K个star，值得充满好奇心的人尝试

	官网地址：http://robolectric.org/

	github  https://github.com/robolectric/robolectric

# 测试实践

1. AS Android项目默认会生成两个test folder

	![](https://i.imgur.com/3aP4FJt.jpg)

2. gradle配置


	![](https://i.imgur.com/rGgtpOi.jpg)

3. 生成类的JUnit测试测试

	![](https://i.imgur.com/zNvEDHO.jpg)
	
	![](https://i.imgur.com/tnHMXuu.jpg)

    注意：JUnit测试不需要连接设备，所以有的框架为了测试遍历会把纯java的功能放在单独的一个module里面

4. 运行，并导出报告

	右键某个类：

	![](https://i.imgur.com/lcZvrIA.jpg)

	也可以右键运行某个package下的所有的测试

	![](https://i.imgur.com/yN2DW5N.jpg)
	
	导出结果：

	![](https://i.imgur.com/T5yI9k6.jpg)

	
	HTML格式的报告like this:

	![](https://i.imgur.com/pw0puJf.jpg)




# Reference

[Espresso 自动化测试框架介绍](https://www.jianshu.com/p/7d257016e129)
