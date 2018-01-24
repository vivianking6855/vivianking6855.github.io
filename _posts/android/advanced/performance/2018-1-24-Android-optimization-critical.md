---
layout: post
title: 性能优化：要点
date: 2018-1-24
excerpt: "性能优化要点"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}



# UI

1. UI不可见时，及时释放资源
2. 页面布局，尽量避免太多层级，避免被VM回收。
3. 页面背景图
    - 在布局和代码中设置背景和图片的时候，如果是纯色，尽量使用color；
    - 如果是规则图形，尽量使用shape画图；
    - 如果是复杂icon，建议使用svg格式
    - 如果svg不满足需求，建议使用9patch图；
    - 如果不能使用9patch的情况下，针对几种主流分辨率的机型进行切图，比如xxhdpi
4. View缓存
    - 在ListView和GridView中，列表中的很多项(convertView)是可以重用的，不需要每次getView就重新生成一项。
    - 页面的绘制其实是很耗时的，findViewById也比较慢。所以不重用View，在有列表的时候就尤为显著了，经常会出现滑动很卡的现象。
    - 推荐使用recyclerview
5. 使用RenderScript、OpenGL来进行复杂的绘图操作 
6. 使用SurfaceView来替代View进行大量、频繁的绘图操作 
7. 自定义View中避免在onDraw方法里面执行复杂的操作，避免创建对象。
8. UI线程只做界面刷新，不做任何耗时操作
    - 耗时操作放在子线程来做 
    - 可以使用Thread+handle，AsyncTask，RxAndroid/RxJava等进行逻辑处理
 
# 线程，进程

1. 使用多进程拆分时请注意，一个空进程也要额外占1.4MB，需评估拆分是否划算？
2. 线程不再需要继续执行的时候要记得及时关闭
3. 开启线程数量不易过多，一般和自己机器内核数一样最好，推荐开启线程的时候，使用线程池。

# 资源回收

1. BraodcastReceiver，ContentObserver，FileObserver，Service的解绑和释放 
    - 以上对象在Activity onDeatory或者某类声明周期结束之后一定要解绑或者close掉
    - 否则Activity类会被system强引用，不会被内存回收。
2. Activity引用WeakReference
    - 不要直接对Activity进行直接引用作为成员变量，如果不得不这么做，请用private WeakReference mActivity来做
    - 相同的，对于Service等其他有自己声明周期的对象来说，直接引用都需要谨慎考虑是否会存在内存泄露的可能。
3. Context使用注意生命周期，避免内存泄漏

    1)	对activity的引用应该控制在activity的生命周期之内。
    
       - 应该尽量避免有appliction进程级别的对象来引用Activity级别的对象，如果有的话也应该在Activity结束的时候解引用。
       - 如不应用applicationContext在Activity中获取资源。Service也一样。
    
    2)	如果不能就考虑使用getApplicationContext或者getApplication；
    
    3)	尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context），即使要使用，也要考虑适时把外部成员变量置空（如上例可以通过把sBackground的callback置空来解决内存泄露的问题）；
    
       也可以在内部类中使用弱引用来引用外部类的变量；
    
    4)	做到在onDestroy中释放资源，如清空对图片等资源有直接引用或者间接引用的数组（使用array.clear();array = null）

4. handler使用完毕及时清理：Activity的onDestroy方法中调用handler.removeCallbacksAndMessages(null);取消所有的消息的处理，包括待处理的消息
5. Cursor及时关闭：当查询完毕后，及时关闭，这样就可以把查询的结果集及时给回收掉。
6. I/O流操作完毕，读写结束，记得关闭。

# 对象创建，访问

1. 如不需要访问对象，请尽量用static method
2. 常量请使用static final来修饰
3. static的合理使用：避免频繁的使用static关键字修饰
    - 一般用来修饰基本数据类型或者轻量级对象，尽量避免修饰集合或者大对象
    - 常用作修饰全局配置项、工具类方法、内部类。
    
    由于static声明变量的生命周期其实是和APP的生命周期一样的（进程级别）。大量的使用的话，就会占据内存空间不释放，积少成多也会造成内存的不断开销，直至挂掉
4. 尽量采用静态内部类，避免潜在的由于内部类而导致的内存泄漏。 
5. 避免创建不必要的对象或内存申请，尽量用原始类型:在java中每个类大约500 bytes code. 每个类实例会占用12-16 bytes以上	
6. 避免临时objects：返回substring取代一个copy string
7. 避免 internal getters/setter (si = mCount (is better than) i = getCount())
8. 避免枚举类型:Enums消耗是静态常量的两倍	
9. 适当使用软引用和弱引用
10. 避免用浮点计算
11. 循环
    - 避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外。
    - 不要在循环当中声明临时变量，不要在循环中捕获异常。
12. 对于那些无法避免需要创建对象的情况，可以考虑对象池模型。
    - 通过对象池来解决频繁创建与销毁的问题。
    - 需要注意结束使用之后，需要手动释放对象池中的对象

13. 使用Android自带的容器，取代HashMap等.比如SparseArray，Pair等，它们都具有更好的性能(Don't use HashMap since it's memory inefficient. Putting a single entry into a HashMap requires 32 bytes.）
    - 使用Android自带的容器SparseArray/SparseBooleanArray/LongSparseArray
    - SparseArray是android里为<Interger,Object>这样的Hashmap而专门写的类,目的是提高效率，其核心是折半查找函数（binarySearch）。
    
            HashMap<Integer, E> hashMap = new HashMap<Integer, E>();
            可以换成
            SparseArray<E> sparseArray = new SparseArray<E>();
14. BitMap隐患
    - Bitmap的不当处理极可能造成OOM，绝大多数情况应用程序OOM都是因这个原因，在操作的时候必须小心。
    - 及时释放recycle。由于Dalivk并不会主动的去回收，需要开发者在Bitmap不被使用的时候recycle掉。
    - 设置一定的压缩率。需求允许的话，应该去对BItmap进行一定的缩放，通过BitmapFactory.Options的inSampleSize属性进行控制。仅只想获得Bitmap的属性，其实并不需要根据BItmap的像素去分配内存，只需在解析读取Bmp的时候使用BitmapFactory.Options的inJustDecodeBounds属性。
    - 建议在加载网络图片的时候，使用软引用或者弱引用并进行本地缓存，推荐使用android-Volley,Picasso、Fresco
15. String/StringBuffer
    - 当有较多的字符创需要拼接的时候，推荐使用StringBuffer。
16. 避免大量使用注解、反射 
17. 如果对于线程安全没有要求，尽量使用线程不安全的集合对象。
18. 使用集合对象，如果事先知道其大小，则可以在构造方法中设置初始大小。


# Service

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
    - startForeground可以避免让Service被强行kill掉，但是同时会带来Service很难会回收的side effect.
    - 非必要强烈建议不要startForeground()启动Service

# 其他

1. 采用内存缓存和磁盘缓存
2. 设定android:largeHeap，使用 android:largeHeap="true"标记 (API Level >= 11) ，
    - 在 AndroidManifest.xml中的 Application节点中声明即可分配到更大的堆内存, 
    - android:largeHeap标记在 Android系统应用中也有广泛的应用 ,比如 Launcher, Browser这些内存大户上均有使用
3.	使用优化后的lib和硬件加速。使用外部lib时注意，确保使用的是为移动环境优化后的lib


[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)


# Reference
