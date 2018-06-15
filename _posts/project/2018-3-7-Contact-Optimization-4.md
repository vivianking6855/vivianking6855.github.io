---
layout: post
title: Contact 优化 - 主页面优化
date: 2018-4-23
excerpt: "Contact 优化 - 主页面优化"
categories: Projects
tags: [Projects]
comments: true
---


* content
{:toc}



# 简介

联系人详情页面主要优化项: 

- 布局优化
- 绘制优化
- Launch time
- 内存优化
- 卡顿优化

# [工具](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)

详情页面的优化会用到下面的工具：

- HierarchyViewer检查layout布局
- Show GPU Overdraw检测Overdraw
- Android Profiler检测CPU,Memory等情况
- LeakCanary 内存泄漏检测工具
- BlockCanary/ [BlockMonitor](http://vivianking6855.github.io/2018/03/27/Android-Block/) 卡顿检测工具

# 环境

- 设备：ZB570TL 
- image: 最新image 
- 联系人 500 + 
- 10笔有头像

# 1. 渲染优化

渲染优化在前面的详情页面有详细步骤，这里不在赘述

## 优化

TODO

## 效果对比

TODO

# 2. Launch time

## 分析 和 优化

1. 使用工具分析模块response time

    - preview launch time：497.40 ms
    - while launch time: 821.20 
    - application launch time: 118.20 

    
    时间统计请参看：200-2 Performance Rule - Contact.xlsx
    
    时间是满足基本要求的

2. 代码逻辑检查

    - 移除TIME_BLOCK相关逻辑，新版不支援

3. 查看多进程情况
   
    判断仅主进程才进入Application的OnCreate中一些初始化
    
    因为remote 百度定位进程不需要初始化这些数据. 

4. 使用MethodTrace监测

    - Application的OnCreate方法耗时细节，我们采用埋点的方式监控
    
        ![](https://i.imgur.com/mcU4HAt.jpg)
        
        可以看出除了一些监测工具的消耗，app本身的一些必要初始化消耗大都在30ms左右，算是正常消耗
    
    - Home页面
    
        首页启动时间点不好掌控，因此我们采用空的Activity过渡的方式搭配CPU Profile来分析
        
         ![](https://i.imgur.com/pUDzaxD.jpg)

        - 系统系统方法的耗时
    
            系统方法的耗时，可以通过渲染优化，逻辑优化，代码优化，算法优化等来优化。
            
            如果来得及，可以先做渲染优化
    
        - 包名过滤 com.**.**

            重点查看10%及以上的方法

            - ViewUtil.isSupportVoLTE(this) 耗时126ms, 13.7%； 有跨进程的操作
            - onResume中的updateViewConfiguration 耗时113，12.4%； 有数据库状态查询和等待的动作


           上面两项几项均放到AsyncTask中执行
           
           例如
           
               class InitAsyncTask extends AsyncTask<Void, Void, Boolean> {
        
                private Reference<MainDialtactsActivity> mActivityRef;
        
                InitAsyncTask(MainDialtactsActivity activity) {
                    mActivityRef = new WeakReference<>(activity);
                }
        
                @Override
                protected Boolean doInBackground(Void... voids) {
                    try {
                        return !(ViewUtil.isCMCCSku() && ViewUtil.isSupportVoLTE(mActivityRef.get().getApplicationContext()));
                    } catch (Exception e) {
                        Log.w(TAG, "InitAsyncTask ex", e);
                    }
        
                    return true;
                }
        
                @Override
                protected void onPostExecute(Boolean aBoolean) {
                    if (mActivityRef == null || mActivityRef.get().isDestroyed()) {
                        return;
                    }
        
                    View conCall = findViewById(R.id.menu_Con_call);
        
                    if (aBoolean) {
                        conCall.setVisibility(View.GONE);
                        ......
                    }
                }
            }

## 效果对比

优化 200ms + 

- 优化前

    ![](https://i.imgur.com/pUDzaxD.jpg)

- 优化后

    ![](https://i.imgur.com/0i2Icws.jpg)


# 3. 内存优化

## 分析和优化

### 1. 接入内存泄漏工具[LeakCanary](https://github.com/square/leakcanary)

一周后，未发现明显Leak问题

### 2. Android Profile监测线程和Service创建情况

    Android Profile监测
    
    -	是否需要创建线程池
    -	AsyncTask是否需要设置并行
    -	Service创建的合理性

检测结果

- 有一处匿名创建Thread的情况，修改放入CacheThreadPool中


## 3. 动态allocation分析

Android Profile record memory查看alloc分配，分析动态allocation是否合理
 

(1) 应用包名过滤: com.**.** [alloc 59K] [Pass]

 - list等都有重用
 - 默认图片也都有重用


(2) 整体分析  [alloc 3.5M]

 - by callstack，共13 thread，查看了200K+的thread        
	- main线程 [2M]，100K以上消耗基本都来自布局加载
	- call log query ThreadPool [758K]
		- 300K+ 是来自CooTek三方库消耗

    到这里可以看出非UI线程消耗基本在1.5M
         
 - by package, 查看100K + 的集合
         
 	main线程和app包名前面分析过了，直接略过。重点看大量或单笔100K+的对象
            
	未发现严重的内存问题

## 4. memory分析内存泄漏

分析退出时的dump memory，查看是否有内存没有及时释放，避免内存泄漏

步骤

- 设置“不保留活动”
- 第一次进入页面，做一些常规操作，GC 2次后dump memory - m1，AS导出 （用作后面的MAT比较）
- 退出重新进入，重复4次，GC 2次后dump meory - m2，AS导出

### AS工具分析

查看是否有多个实例存在的情况（因为我们退出/重进了多次）

按照包名排序查看，HomeActivity没有泄漏的情况，都有成功销毁
	
![](https://i.imgur.com/gkdEtXg.jpg)
	
还有model这里，有四个Reference，从分析来看他们都是指向同一个实例。之所有会有这么多是因为有很多内部类持有了外类的引用
	
![](https://i.imgur.com/l7ssEfs.jpg)

注意哈，这里有两个Application对象和两个TestLaunchActivity对象，是因为app有两个进程

分析结果：暂没有发现严重的内存泄漏
	
### MAT分析内存泄漏

转MAT格式
	    
	hprof-conv heap-original.hprof heap-converted.hprof

我们先采用明确方式，再采用传统方式，最后做进阶分析

1. 明确方式

2. 传统方式

3. 进阶分析


## 效果对比

待补充

# 测试重点

1. 主页面加载
2. 联系人列表，编辑
3. call log列表，编辑

# Reference

[性能优化 - 目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

[性能优化 - Android Studio & MAT 如何分析内存](http://vivianking6855.github.io/2018/05/04/Android-optimization-AS-MAT/)