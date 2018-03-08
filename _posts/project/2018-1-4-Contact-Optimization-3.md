---
layout: post
title: Contact 优化 - detail页面优化
date: 2018-1-4
excerpt: "Contact 优化 - detail页面优化"
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
- 内存优化
- Launch time

# [工具](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)

详情页面的优化会用到下面的工具：

- HierarchyViewer检查layout布局
- Show GPU Overdraw检测Overdraw
- GPU呈现模式分析UI渲染效率
- Android Profiler检测CPU,Memory等情况
- LeakCanary 内存泄漏检测工具
- BlockCanary 卡顿检测工具

# 环境

- 设备：ZB570TL 
- image: 最新image 
- 单个联系人详情包含所有类别的信息
- 单个联系人call log 10笔

# 优化分析和实践
  
## 1. Layout优化

我们的目标：布局尽量扁平化，移除非必需的UI组件和不必要的嵌套。

避免布局是否过于复杂，来减少Measure，Layout的计算时间，优化CPU性能

优化前建议先用Layout Inspector工具查看清楚布局，方便后面的优化。

![修改前](https://i.imgur.com/cuDdQs3.jpg)

绿色的是系统层的view，自己的layout从蓝色标注开始

![](https://i.imgur.com/RY5FcYk.png)

### 分析

我们首先使用HierarchyViewer检测layout层级

![](https://i.imgur.com/SAT0iFR.jpg)

可以看到布局嵌套层数较多，而且有部分view的Measure, Layout, Draw非常慢（红色的点）

这说明我有太多的嵌套，布局层次太深，有些View的绘制可能有问题（比如复杂计算）

### 实践

来看看布局文件如何修改

1.  ColorfulLinearLayout

    从Code中可以看到ColorfulLinearLayout 和 RelativeLayout是重复的.   

    ColorfulLinearLayout加入是因为要在4.4上支持透明status bar，现在可以移除
   
    减少层级： 1层；减少控件: 1个 

2. Cover和tab navigator 布局

    Cover布局的层级最多有4层
       
    - 整个Cover layout优化至1层
    - asus_contact_detail_cover布局RelativeLayout中仅有一个控件ImageView，可以移除RelativeLayout，仅保留ImageView，减少一层   

    tab navigator布局层级最多有3层
    
    - 修改layout可以直接放到root layout里面，即节省3层
    - Image+TextView换成TextView(drawableTop属性设置图片)，减少一半的控件。
   
    减少层级: 7层；减少控件: 2个        
    
3. bottom布局

    bottom的部分布局Image+TextView换成TextView(drawableTop属性设置图片)，减少一半的控件
    
    减少控件: 10个       
    
4. detail Fragment布局
    - ScrollView + TextView layout移除ScrollView，TextView自己支持滚动
    - FragmentLayout和LinearLayout重叠，移除FragmentLayout
    - ListViewitem局层级过多:number detail层级最多6层，可以优化至2层
    - 分割线单个LinearLayout改为单个View

    减少层级: 8层；减少控件: 1个；优化控件: 1个    

5. calllog Fragment

    - FragmentLayout和RelativeLayout重叠，移除FragmentLayout
    - call log header 2层，直接移除：layout设置gone, 而且code中没有设置显示，
    - call log list item层级最多6层，可以优化到2层
    - RelativeLayout优化为LinearLayout

    减少层级: 6层；减少控件: 6个；优化控件: 1个

## 2. Overdraw优化（过度绘制优化）

有时因为UI设计的需要，有些过度绘制是不可避免的。

比较正常基本都是在1x-2x范围内，例如Google now页面。如果超过3x，过渡绘制是很严重的了。

我们的目标：尽可能达控制在0-2层Overdraw

### 分析

使用工具“Show GPU view updates”检测Overdraw

- 没有任何优化前的Overdraw

    ![](https://i.imgur.com/za5OtbZ.png)

    可以看到很多都是3x，4x+ 的Overdraw，说明过渡重绘非常严重。

- 经过layout优化后的Overdraw

    ![](https://i.imgur.com/BhiwLGh.png)

    可以看到基本都是1x，2x Overdraw. 

我们的UI设计要求：

- top要有图片背景
- list item需要白色背景，空的部分需要灰色背景
- bottom底部也是灰色背景
- Activity的默认白色背景

有些地方不可避免的会有2层Overdraw

备注：检查code是否可以优化的空间。可以用Layout Inspector工具辅助查看layout背景情况(properties中的bg_state_mUseColor)

### 实践

从设计来看可以优化的部分：

1. 移除listview 背景，直接使用Activity DecorView的默认白色

    空余的灰色背景加入一个View来替代

    减少listview一层背景

## 3. Launch time

### 分析

Activity launch time优化主要是检查OnCreate中的耗时逻辑，将耗时的任务延迟或放到线程中，来优化launch time

优化前需要熟悉launch的加载逻辑。Launch time我们参照从OnCreate到onWindowFocusChanged的时间差。

现在的时间差约：581ms

### 实践

从加载逻辑来分析，看哪些可以优化

- 移除UI有重复刷新
- 背景和头像背景放到AsyncTask中加载
- 移除部分无用数据加载

## 4. 内存优化

### 1) Memory Leak检测

整合LeakCanary，开始Memory Leak检测

测试前若AndroidManifest中有设置android:largeHeap="true"，建议先移除。

检测结果：3台设备，正常使用2周，未弹出Memory Leak警告

### 2) 代码逻辑和代码优化

- Bitmap优化：Bitmap优化通常包括缓存和采样率优化。联系人详情中没有大量的图片，因此这里对采样率做优化
    - 压缩cover背景图（inSampleSize），防止加载图片时内存溢出
    - 移除中间一笔无用Bitmap创建
- 代码优化
    - 数组copy优化为System.arraycopy，Native层的arraycopy速度非常快
    - 优化UpdateGroupTask避免memory leak：加入Activity isDestroyed判断，即使释放资源和避免无效的数据加载
    - 移除无效code 

java code文件减少size: 10.6 MB - 10.1 MB = 0.5M

PS: 代码优化可以参看：[性能优化要点](http://vivianking6855.github.io/2018/01/24/Android-optimization-critical/)。建议代码优化时，参照AndroidStudio的Lint工具提醒

### 3) Android Profiler工具

设置Contact 详情 10 item，call log 10笔

#### 查看Memory Allocation

从主页面跳转到详情页面，查看Memory分配情况。是否有不合理的Memory Allocation

![](https://i.imgur.com/t96Zm6j.jpg)

- byte[]
    - getView内部对象创建也是很耗资源的，跟viev item无关的建议都移到外部去
    - 将用户自定义头像的图片和背景背景采样率设为4，将默认头像背景采样率设置2
    - 在callLogDetailFragment的getData方法中，移除无必要的多次new simpleDateFormat和new Date操作
    - 移除无必要的instanceof 操作
- long[] 
    - remove stream items 
- char[], String
    - isSimActive调用几十次(detail item越多调用次数越多)，每次都会allocate 532多byte.优化为调用一次
    - isSupportVoLTE调用几十次(detail item越多调用次数越多)，每次都会allocate 228多byte.优化为调用一次
    - QueryCouponAsyncTask不必要的task加载，导致detail adapter刷新异常。移除此项Task
    - ArrayList移到判断条件内部，有需要的时候在创建
    - 移除AsusRedPointNotificationDualPanelHelper，小红点功能CN不支援

#### 页面完全退回Memory检测
  
  其实LeakCanary已经可以帮忙检测出部分，如果是不放心这里还可以再详尽的检查一次
   
  原则上需要释放无用资源。 可以设置开发者选项开启“不保留活动”，强制GC后，分析HeapDump

# 优化效果对比

## 1. layout优化对比

优化成果：

- 减少层级: 22层
- 减少控件: 20个
- 优化控件: 2个

优化前后层级图对比：（后者为优化后）

- 全景图对比

    ![修改前](https://i.imgur.com/cuDdQs3.jpg)
    
    ![修改后](https://i.imgur.com/FAiXuaP.jpg)
    
各个子项的Layout对比可以查看：F:\Manager\Contact\Optimization\datum\detail\layout

## 2. Overdraw优化对比

- 优化前：3x-4x 过渡绘制
- 优化后：0-2x 过渡绘制

基于现有UI设计要求，已达到最优Overdraw

优化前后Overdraw效果图对比（后者为优化后)

![优化前](https://i.imgur.com/za5OtbZ.png)

![优化后](https://i.imgur.com/2J66BcQ.png)

## 3. Launch Time优化 

优化前后 Launch time of Contact detail

- 优化前：581ms
- 优化后：499ms

共节省14% launch time, 共82ms 

## 4. CPU和Memory效果对比

经过layout，Overdraw，内存等优化，CPU和Memory的优化效果如下：

- 优化前：memory 117.62M，cpu 15%
- 优化后：memory 105.63M, cpu 13.1%

Memory减少12M+,	CPU降低约2%

这里测试的联系人详情内容10笔，call log 10笔。若内容和calllog增加，memory和cpu会减少更多

Memory优化前后效果图对比（后者为优化后)

![Memory优化前](https://i.imgur.com/Sv4rYxy.jpg)  ![Memory优化后](https://i.imgur.com/oeDpgM7.jpg)

CPU优化前后效果图对比（后者为优化后)

![CPU优化前](https://i.imgur.com/Gyd22SC.jpg)  ![CPU优化后](https://i.imgur.com/6p3NRQu.jpg)
 
# 注意事项

- layout优化层级变动时，可能会导致UI，特别是背景有误。layout优化完成后要对比各种scenario时的UI
- image压缩时，要注意是否失真不符合预期效果

# 测试重点

1. detail页面UI展示测试
2. detail页面的逻辑操作
3. detai页面多笔item加载场景
4. call log多笔item加载场景
5. 联系人详情增加多笔item和call log场景