---
layout: post
title: 性能优化（一）：方法概述
date: 2017-2-27
excerpt: "方法概述"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

关于性能优化的问题，主要关注的有：

- 内存
- CPU
- 耗电
- 卡顿
- 渲染
- 进程存活率等

性能优化注意事项：

- 不要过早的做性能优化，app先求能用再求好用。在需求都还没完成的时候，花大量时间在优化上是本末倒置的
- 优化要用实际数据说话，建议借助测试工具进行检测
    - 网易的Emmagee
    - 腾讯的GT和APT
    - 科大讯飞的iTest
    - Google的Battery Historian

合理优化，数据量化


# 一、 布局优化

布局优化的思想：尽量减少层级和无用的控件。 

1. Layout 设计优化

    在布局设计时，就应该考虑最优化思想。下面列出一些常用的技巧：

    - 有选择地使用性能较低的ViewGroup.比如不嵌套的情况下，用LinearLayout和FrameLayout代替RelativeLayout.
        - RelativeLayout功能比较复杂，布局过程需要花费更多的CPU时间
        - 但是如果要LinearLayout嵌套来代替RelativeLayout，还是建议用RelativeLayout。因为嵌套同样会降低程序的性能  
    - 使用include实现布局重用，避免代码重复
    - 使用merge减少布局层级结构
    - 使用ViewStub实现延时加载
    - 在TextView中使用Compound drawable，取代ImageView + TextView
    - 使用LinearLayout自带的分割线: android:divider=""

2. Hierarchy Viewer工具优化布局
   

详见：[性能优化（四）Google典范之Render实践中Layout优化章节](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 二、 绘制优化

1. 移除不必要的background
2. 用clipRect优化
3. onDraw方法要避免执行大量的操作

详见：[性能优化（四）Google典范之Render实践中Overdraw章节](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 三、 内存泄漏优化

内存泄漏是在开发过程需要重视的问题。内存泄漏优化分两个方面：

- 开发过程中避免写出有内存泄漏的代码
- 大量使用类似Material Design的风格，不仅安装包可以变小，还可以减少内存的占用，渲染性能与加载性能都会有一定的提升。
- 通过一些分析工具如Android Studio自动Android Monitor，[leakcanary](https://github.com/square/leakcanary)，FindBugs等

详见[Android性能优化 （二）内存 OOM](http://vivianking6855.github.io/tag/#Android%20%E8%BF%9B%E9%98%B6-ref)

内存优化并不就是说程序占用的内存越少就越好。

如果因为想要保持更低的内存占用，而频繁触发执行gc操作，在某种程度上反而会导致应用性能整体有所下降。需要综合考虑权衡。

# 四、 响应速度优化

响应速度优化核心思想是避免在主线程中做耗时的操作。耗时的操作应该放到异步任务中执行。

# 五、 ListView优化

构造Adapter时，没有使用缓存的 convertView，可以使用Android最新组件RecyclerView,替代ListView来避免

# 六、 Bitmap优化

主要是BitmapFactory.Options 和缓存。详见[Android Bitmap的加载和Cache](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

# 七、 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread. 控制最大并发数等。详见下面两篇blog~

[Android的线程和线程池，线程同步](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

# 八、 耗电优化

电量其实是目前手持设备最宝贵的资源之一，大多数设备都需要不断的充电来维持继续使用。

Purdue University研究了最受欢迎的一些应用的电量消耗：

- 平均只有30%左右的电量是被程序最核心的方法例如绘制图片，摆放布局等等所使用掉的
- 剩下的 70%左右的电量是被上报数据，检查位置信息，定时检索后台广告信息所使用掉的。

如何平衡这两者的电量消耗，就显得非常重要了。有下面一些措施能够显著减少电量的消耗：

- 我们应该尽量减少唤醒屏幕的次数与持续的时间，使用WakeLock来处理唤醒的问题，能够正确执行唤醒操作并根据设定及时关闭操作进入睡眠状态。
- 某些非必须马上执行的操作，例如上传歌曲，图片处理等，可以等到设备处于充电状态或者电量充足的时候才进行。
- 触发网络请求的操作，每次都会保持无线信号持续一段时间，我们可以把零散的网络请求打包进行一次操作，避免过多的无线信号引起的电量消耗。
    - 关于网络请求引起无线信号的电量消耗，还可以参考[这里](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)
- App有电量消耗过多的问题，我们可以使用[JobScheduler](http://hukai.me/android-training-course-in-chinese/background-jobs/scheduling/index.htm) API来对一些任务进行定时处理
    - 例如我们可以把那些任务重的操作等到手机处于充电状态，或者是连接到WiFi的时候来处理。
- 使用WakeLock或者JobScheduler唤醒设备处理定时的任务之后，一定要及时让设备回到初始状态。
- 每次唤醒无线信号进行数据传递，都会消耗很多电量，它比WiFi等操作更加的耗电，[详情](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)


我们可以通过手机设置选项找到对应App的电量消耗统计数据。我们还可以通过Android 5.0发布的Battery History Tool来查看详细的电量消耗。

请关注程序的电量消耗，用户可以通过手机的设置选项观察到那些耗电量大户，并可能决定卸载他们。所以尽量减少程序的电量消耗是非常有必要的。

# 九、 性能优化小建议

性能优化的小建议：

1.	如无必要，service不要一直在跑

    如果应用程序当中需要使用Service来执行后台任务的话，请一定要注意只有当任务正在执行的时候才应该让Service运行起来。
    另外，当任务执行完之后去停止Service的时候，要小心Service停止失败导致内存泄漏的情况。
    当我们启动一个Service时，系统会倾向于将这个Service所依赖的进程进行保留，这样就会导致这个进程变得非常消耗内存。
    并且，系统可以在LRU cache当中缓存的进程数量也会减少，导致切换应用程序的时候耗费更多性能。
    严重的话，甚至有可能会导致崩溃，因为系统在内存非常吃紧的时候可能已无法维护所有正在运行的Service所依赖的进程了。

    为了能够控制Service的生命周期，Android官方推荐的最佳解决方案就是使用IntentService，这种Service的最大特点就是当后台任务执行结束后会自动停止，从而极大程度上避免了Service内存泄漏的可能性。
    关于IntentService更加详细的用法讲解，可以参考《第一行代码——Android》的9.5.2节。
    让一个Service在后台一直保持运行，即使它并不执行任何工作，这是编写Android程序时最糟糕的做法之一。
    所以Android官方极度建议开发人员们不要过于贪婪，让Service在后台一直运行，这不仅可能会导致手机和程序的性能非常低下，而且被用户发现了之后也有可能直接导致我们的软件被卸载。

2. 如无必要，不要使用startForeground() 来启动 service 

    startForeground可以避免让Service被强行kill掉，但是同时会带来Service很难会回收的side effect.
    非必要强烈建议不要startForeground()启动Service

3.  UI不可见时，及时释放资源
4.	页面布局，尽量避免太多层级，避免被VM回收。
5.	使用多进程，UI与work进程分离。长时间的操作要放到worker线程以减少ANR。拆分时请注意，一个空进程也要额外占1.4MB，需评估拆分是否划算？
6.	线程
    - 线程不再需要继续执行的时候要记得及时关闭，开启线程数量不易过多，一般和自己机器内核数一样最好，推荐开启线程的时候，使用线程池。
7.	使用优化后的lib和硬件加速。使用外部lib时注意，确保使用的是为移动环境优化后的lib
8.	BraodcastReceiver，ContentObserver，FileObserver，Service的解绑和释放 
    
    BraodcastReceiver，ContentObserver，FileObserver，Service在Activity onDeatory或者某类声明周期结束之后一定要解绑或者close掉，
    否则这个Activity类会被system强引用，不会被内存回收。
    
9.	Activity引用WeakReference
    
    不要直接对Activity进行直接引用作为成员变量，如果不得不这么做，请用private WeakReference mActivity来做，相同的，对于Service等其他有自己声明周期的对象来说，直接引用都需要谨慎考虑是否会存在内存泄露的可能。正确的用法如下：
 
10.	Context 不能保持长生命周期引用

    例如：不能这样使用sBackground的生命周期比Activity要长

    1)	对activity的引用应该控制在activity的生命周期之内。
    
       - 应该尽量避免有appliction进程级别的对象来引用Activity级别的对象，如果有的话也应该在Activity结束的时候解引用。
       - 如不应用applicationContext在Activity中获取资源。Service也一样。
    
    2)	如果不能就考虑使用getApplicationContext或者getApplication；
    
    3)	尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context），即使要使用，也要考虑适时把外部成员变量置空（如上例可以通过把sBackground的callback置空来解决内存泄露的问题）；
    
       也可以在内部类中使用弱引用来引用外部类的变量；
    
    4)	做到在onDestroy中释放资源，如清空对图片等资源有直接引用或者间接引用的数组（使用array.clear();array = null）

11.	设定android:largeHeap 

    使用 android:largeHeap="true"标记 (API Level >= 11) ，在 AndroidManifest.xml中的 Application节点中声明即可分配到更大的堆内存, 
    android:largeHeap标记在 Android系统应用中也有广泛的应用 ,比如 Launcher, Browser这些内存大户上均有使用

12.	如不需要访问对象，请尽量用static method"
13.	使用Android自带的容器，取代HashMap等

    比如SparseArray，Pair等，它们都具有更好的性能

    Don't use HashMap since it's memory inefficient. Putting a single entry into a HashMap requires 32 bytes.
    使用Android自带的容器SparseArray/SparseBooleanArray/LongSparseArray
    SparseArray是android里为<Interger,Object>这样的Hashmap而专门写的类,目的是提高效率，其核心是折半查找函数（binarySearch）。
    在Android中，当我们需要定义
    
    HashMap<Integer, E> hashMap = new HashMap<Integer, E>();

    我们可以使用如下的方式来取得更好的性能.

    SparseArray<E> sparseArray = new SparseArray<E>();

14.	避免枚举类型
    
    Enums消耗是静态常量的两倍	
    
15.	避免创建不必要的对象或内存申请，尽量用原始类型

    在java中每个类大约500 bytes code. 每个类实例会占用12-16 bytes以上	

16.	避免临时objects：返回substring取代一个copy string
17.	避免 internal getters/setter (si = mCount (is better than) i = getCount())
18.	避免使用bitmap
19.	避免用浮点计算
20. 避免创建过多的对象
21. 常量请使用static final来修饰
22. 适当使用软引用和弱引用
23. 采用内存缓存和磁盘缓存
24. 尽量采用静态内部类，避免潜在的由于内部类而导致的内存泄漏。
25. 避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外
26. 自定义View中避免在onDraw方法里面执行复杂的操作，避免创建对象。
27. 对于那些无法避免需要创建对象的情况，可以考虑对象池模型。
    - 通过对象池来解决频繁创建与销毁的问题。
    - 需要注意结束使用之后，需要手动释放对象池中的对象
28. static的合理使用，避免频繁的使用static关键字修饰
    - 一般用来修饰基本数据类型或者轻量级对象，尽量避免修饰集合或者大对象，常用作修饰全局配置项、工具类方法、内部类。
    - 由于static声明变量的生命周期其实是和APP的生命周期一样的（进程级别）。大量的使用的话，就会占据内存空间不释放，积少成多也会造成内存的不断开销，直至挂掉。
29. BitMap隐患
    - Bitmap的不当处理极可能造成OOM，绝大多数情况应用程序OOM都是因这个原因，在操作的时候必须小心。
    - 及时释放recycle。由于Dalivk并不会主动的去回收，需要开发者在Bitmap不被使用的时候recycle掉。
    - 设置一定的压缩率。需求允许的话，应该去对BItmap进行一定的缩放，通过BitmapFactory.Options的inSampleSize属性进行控制。仅只想获得Bitmap的属性，其实并不需要根据BItmap的像素去分配内存，只需在解析读取Bmp的时候使用BitmapFactory.Options的inJustDecodeBounds属性。
    - 建议在加载网络图片的时候，使用软引用或者弱引用并进行本地缓存，推荐使用android-Volley,Picasso、Fresco
30. 页面背景图
    - 在布局和代码中设置背景和图片的时候，如果是纯色，尽量使用color；如果是规则图形，尽量使用shape画图；如果稍微复杂点，可以使用9patch图；如果不能使用9patch的情况下，针对几种主流分辨率的机型进行切图。
31. View缓存
    - 在ListView和GridView中，列表中的很多项(convertView)是可以重用的，不需要每次getView就重新生成一项。
    - 页面的绘制其实是很耗时的，findViewById也比较慢。所以不重用View，在有列表的时候就尤为显著了，经常会出现滑动很卡的现象。
    - 推荐使用recyclerview
32. handler 清理
    - 在Activity的onDestroy方法中调用handler.removeCallbacksAndMessages(null);取消所有的消息的处理，包括待处理的消息；
33. Cursor及时关闭
    - 在查询SQLite数据库时，会返回一个Cursor，当查询完毕后，及时关闭，这样就可以把查询的结果集及时给回收掉。
34. I/O流
    - I/O流操作完毕，读写结束，记得关闭。
35. String/StringBuffer
    - 当有较多的字符创需要拼接的时候，推荐使用StringBuffer。

<br/><br/>


# Reference

<br>



> 性能优化专题

[性能优化（一）：方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)

[性能优化（二）内存管理 & Memory Leak & OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)

[性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)

[性能优化（四）Google典范之Render实践](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

> 《Android开发艺术探索》

> [Android内存优化之OOM](http://www.csdn.net/article/2015-09-18/2825737/5)

> [Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

> [优达学城 Android 系统性能 Video](https://cn.udacity.com/course/android-performance--ud825)

> [Google Android Performance Patterns YouTube ](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

> [性能优化系列总篇](http://www.trinea.cn/android/database-performance/)

> [Android性能优化全方面解析](http://blog.csdn.net/sw950729/article/details/72124008)