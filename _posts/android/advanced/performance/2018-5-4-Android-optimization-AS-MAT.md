---
layout: post
title: 性能优化 - Android Studio & MAT 如何分析内存
date: 2018-5-4
excerpt: "性能优化 - Android Studio & MAT 如何分析内存"
categories: Android 性能优化
tags: [Android 性能优化]
comments: true
---


* content
{:toc}



# 简介

因为AS的升级，很多的网上的资料都已经过时，这里这基于现有最新的AS，来总结如何搭配MAT对Memory进行分析和优化。

环境：Android Studio 3.1.2（2018/5/4)


# 导图

![](https://i.imgur.com/K9evyS1.png)

# 1. 接入LeakCanary

在开始内存分析前，建议先接入内存泄漏工具[LeakCanary](https://github.com/square/leakcanary)，先解决一些明显的内存问题。

# 2. Android Profile监测线程和Service创建情况

在开始分析dump和allocation前，我们先使用Android Profile监测当前模块的线程和Service创建情况。

很多不规范的写法会在代码中创建很多的匿名线程和无用Service. 我们需要检查
    
- 是否需要创建线程池，代替多个Thread
- AsyncTask是否需要设置并行（默认是串行）
- Service创建的合理性

# 3. 动态allocation分析

Android Profile record memory查看alloc分配，分析动态allocation是否合理
 
- 应用包名过滤: com.**.** [alloc ? K] [Pass ?]
 
	比如：list，默认图片等是否有重用
- 整体分析  [alloc ?] [Pass ?]
	- by callstack，可以查看200K+的thread memory消耗      
    - by package, 可以查看100K + 的集合

# 4. destroy memory分析

分析destroy后的memory，是分析内存泄漏的一个常用的入手点。

设置“不保留活动”，分析退出时的dump memory，查看是否有内存没有及时释放，避免内存泄漏

操作步骤

 - 设置“不保留活动”
 - 第一次进入页面，做一些常规操作，GC 2次后dump memory - m1，AS导出 （用作后面的MAT比较）
 - 退出重新进入，重复4次，GC 2次后dump meory - m2，AS导出

## AS工具分析

查看是否有多个实例存在的情况（因为我们退出/重进了多次）
	
- 按照包名排序查看主要的对象，比如Activity，Fragment，数据缓存类等
	
## MAT分析

转MAT格式
	    
	hprof-conv heap-original.hprof heap-converted.hprof

为了后面描述的直观性，这里我们在MainActivity中加入Leak代码如下：

	    public void testMemoryLeak(View v) {
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                try {
	                    Thread.sleep(60 * 60 * 1000);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	
	            }
	        }, "MemoryLeakThread").start();
	    }


1. 打开Leak Suspects，查看MAT给出怀疑对象
			
	- 可以作为参看，不过通常用处不是特别大
	- 常见的几项如下，其实大部分可能来自res资源和类加载的消耗
		- android.graphics.Bitmap
		- java.lang.String
		- java.lang.Class


2. 目标方式（适合于目标很明确的情形）

	比如查找自己负责的Activity，或者特定包名。可以通过包名或者Class筛选，OQL搜索都可以快速定位到
				
		例如: select * from instanceof android.app.Activity
	
	然后分析引用链，查找root cause. 引用链分析步骤通常都是：
			
	- List Objects (incomming，outgoing)
	- 选择多个对象Merge Shortest Path to GCRoots；单个对象Paths to GC Roots(exclude all phantom/weak/soft etc.references)
	- 查看引用链，分析具体的引用为何没有被释放，并进行修复

	分析的结果：有Thread对MainActivity持有强引用导致，对象无法释放，出现内存泄漏

	![](https://i.imgur.com/ezKeTbs.jpg)

3. 传统方式（无具体目标）
				
	当目的不明确时，我们可以先AS监控app的Memory如果出现Memory一直增长的情况。很可能有Memory性能问题。
	
	我们可以先从自己的package和RetainedHeap较大的Object出发。打开Histogram（每个类的实例数）
	
	- 方式一： by package
	
		查看自己package内的可疑引用链: Merge Shortest Path to GCRoots 或者List Objects (incomming)，再Path to GC Roots也可以
	
	- 方式二：RetainedHeap较大的Object				
	
		分析RetainedHeap较大的Object, 分析引用链（方式同上）
	
		步骤图解如下：
		
		![](https://i.imgur.com/xWQZOXX.jpg)
		
		![](https://i.imgur.com/H3F5HHP.jpg)
		
		一个empty的app的大部分消耗基本都在Resource加载，如下图：
		
		![](https://i.imgur.com/4xEyyhV.jpg)
	
	- 方式三 对比分析
	
		在没有方向时，对比分析也可以帮助我们很快的定位问题。
	
		我们dump两笔memory：一次退出和多次退出的hprof
	
		使用Compare Basket比较（Window ->Navigation History 把两笔Histogram加入Compare Basket）
	
		可以先加入过滤（包名或其他），找到多笔实例分析。例如
	
		![](https://i.imgur.com/tVn0ZTm.jpg)
	
		![](https://i.imgur.com/ezKeTbs.jpg)
		    
# 5. [dump MAT 进阶分析](http://www.lightskystreet.com/2015/09/01/mat_usage/)

深入分析主要是针对一些容易出现性能问题的对象，例如Bitmap, Hash集合，empy集合等。

1. Bitmap

	Bitmap毫无疑问是Memory的大户，分析步骤如下：

	- 打开Dominator Tree（对象关系树） 
	- filter过滤android.graphics.Bitmap
	- Merge Shortest Path to GCRoots 分析引用链

2. 集合使用率分析（部分内存摘录自参考[2])

	集合在开发中会经常使用到，如何选择合适的数据结构的集合，初始容量是多少（太小，可能导致频繁扩容），太大，又会开销跟多内存。
	当这些问题不是很明确时或者想查看集合的使用情况时，可以通过MAT来进行分析。
	
	- 打开Histogram
	- 包名排序，定位com--android

		![](https://i.imgur.com/AyOP5NL.jpg)

	- 右键：Show Retained Set 查找android目录下的依赖集合
	- 筛选指定的Object（Hash Map，ArrayList）并按照大小进行分组

		![](http://www.lightskystreet.com/img/mat/collection_usage_3.png)

		- 查看指定类的Immediate dominators

			![](http://www.lightskystreet.com/img/mat/collection_usage_4.png)
		
		- Collections fill ratio

			这种方式只能查看那些具有预分配内存能力的集合，比如HashMap，ArrayList。计算方式：”size / capacity”

			![](http://www.lightskystreet.com/img/mat/collection_usage_fill_radio_1.png)
			
			![](http://www.lightskystreet.com/img/mat/collection_usage_fill_radio_2.png)

2. Hash等相关性能分析

	当Hash集合中过多的对象返回相同Hash值的时候，会严重影响性能（Hash算法原理自行搜索，不过Java 1.8 HashMap引入了红黑平衡树，性能有所提高）

	这里来查找导致Hash集合的碰撞率较高的罪魁祸首

	- Map Collision Ratio
	
		检测每一个HashMap或者HashTable实例并按照碰撞率排序
		碰撞率 = 碰撞的实体/Hash表中所有实体
		
		![](http://www.lightskystreet.com/img/mat/map_collision_1.png)

	- 查看Immediate dominators

		![](http://www.lightskystreet.com/img/mat/map_collision_2.png)

	- 通过HashEntries查看key value

		![](http://www.lightskystreet.com/img/mat/map_collision_3.png)


	Array等其它集合分析方法类似


3. 未使用的集合分析

	- 通过OQL快速查询empty并且未修改过的集合

			select * from java.util.ArrayList where size=0 and modCount=0
			select * from java.util.HashMap where size=0 and modCount=0
			select * from java.util.Hashtable where count=0 and modCount=0

	- 可以全部选中，Immediate dominators(查看引用者)，查看浪费了多少内存

		![](http://www.lightskystreet.com/img/mat/empty_list_1.png)

		![](https://i.imgur.com/usO1Zq1.jpg)


# Reference

[性能优化 - 目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[1. MAT使用教程](http://blog.csdn.net/itomge/article/details/48719527)

[2. MAT - Memy Analyzer Tool 使用进阶](http://www.lightskystreet.com/2015/09/01/mat_usage/ )
