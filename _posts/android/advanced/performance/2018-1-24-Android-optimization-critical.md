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


# 逻辑优化

是理清程序逻辑，减少不必要的操作。

# 需求优化

合理评估需求

---

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
9. 在任何时候都请避免调用requestLayout()的方法，因为一旦调用了requestLayout，会导致该layout的所有父节点都发生重新layout的操作。
    - 如果无法避免重复layout，应该尽量保持View Hierarchy的层级比较浅，这样即使发生重复layout，也不会因为布局的层级比较深而增大了重复layout的倍数。
    - 可以使用Systrace来跟踪特定的某段操作，如果发现了疑似丢帧的现象，可能就是因为重复layout引起的

---

# 线程，进程及通讯

1. 使用多进程拆分时请注意，一个空进程也要额外占1.4MB，需评估拆分是否划算？
2. 线程不再需要继续执行的时候要记得及时关闭
3. 开启线程数量不易过多，一般和自己机器内核数一样最好，推荐开启线程的时候，使用线程池。
4. LocalBroadcastManager代替普通BroadcastReceiver，效率和安全性都更高

---

# 资源回收

1. BraodcastReceiver，ContentObserver，FileObserver，Service的解绑和释放 
    - 以上对象在Activity onDeatory或者某类声明周期结束之后一定要解绑或者close掉
    - 否则Activity类会被system强引用，不会被内存回收。
2. SoftReference、WeakReference相对正常的强应用来说更有利于系统垃圾回收
    - 不要直接对Activity进行直接引用作为成员变量，如果不得不这么做，请用private WeakReference mActivity来做
    - 相同的，对于Service等其他有自己声明周期的对象来说，直接引用都需要谨慎考虑是否会存在内存泄露的可能。
3. Context使用注意生命周期，避免内存泄漏
    - 对activity的引用应该控制在activity的生命周期之内。
        - 应该尽量避免有appliction进程级别的对象来引用Activity级别的对象，如果有的话也应该在Activity结束的时候解引用。
        - 如不应用applicationContext在Activity中获取资源。Service也一样。
    - 如果不能就考虑使用getApplicationContext或者getApplication
    - 尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context），即使要使用，也要考虑适时把外部成员变量置空（如上例可以通过把sBackground的callback置空来解决内存泄露的问题）；
    - 也可以在内部类中使用弱引用来引用外部类的变量
    - 做到在onDestroy中释放资源，如清空对图片等资源有直接引用或者间接引用的数组（使用array.clear();array = null）
4. handler使用完毕及时清理：Activity的onDestroy方法中调用handler.removeCallbacksAndMessages(null);取消所有的消息的处理，包括待处理的消息
5. Cursor及时关闭：当查询完毕后，及时关闭，这样就可以把查询的结果集及时给回收掉。
6. I/O流操作完毕，读写结束，记得关闭。
7. Application，Activity，Fragment，Service，Content Provider的onTrimMemory()中释放不必要内存
    - 当系统内存达到某些条件的时候，所有正在运行的应用都会收到这个回调。需要尽快释放当前应用的非必须内存资源（会传入不同的内存使用情况）
    - Android 4.4开始，ActivityManager提供了isLowRamDevice()的API，通常指的是Heap Size低于512M或者屏幕大小<=800*480的设备
8. View会保持Activity的引用，Activity以及view的泄漏是非常严重的，为了避免出现泄漏，请特别留意以下的规则：
    - 避免使用View异步回调，如果一定要的话请注意及时cancel任务和判空。因为异步回调被执行的时间不确定，很有可能发生在activity已经被销毁之后，这不仅仅很容易引起crash，还很容易发生内存泄露。
    - 避免使用Static对象View.因为static的生命周期过长，使用不当很可能导致leak，在Android中应该尽量避免使用static对象。
    - 避免把View添加到没有清除机制的容器里面。假如把view添加到 WeakHashMap，如果没有执行清除操作，很可能会导致泄漏。

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

# 对象创建，访问

- 如不需要访问对象，请尽量用static method
- static的合理使用：避免频繁的使用static关键字修饰
    - 一般用来修饰基本数据类型或者轻量级对象，尽量避免修饰集合或者大对象
    - 常用作修饰全局配置项、工具类方法、内部类。
    
    由于static声明变量的生命周期其实是和APP的生命周期一样的（进程级别）。大量的使用的话，就会占据内存空间不释放，积少成多也会造成内存的不断开销，直至挂掉
- 尽量采用静态内部类，避免潜在的由于内部类而导致的内存泄漏。 
- 避免创建不必要的对象或内存申请，尽量用原始类型:在java中每个类大约500 bytes code. 每个类实例会占用12-16 bytes以上	
- 避免临时objects：返回substring取代一个copy string
- 避免 internal getters/setter (si = mCount (is better than) i = getCount())
- 避免[枚举类型](http://blog.csdn.net/qiuchengjia/article/details/52910948)
    - Enums消耗是静态常量的两倍。
    - 建议用[@IntDef,@StringDef](https://noobcoderblog.wordpress.com/2015/04/12/java-enum-and-android-intdefstringdef-annotation/)等代替
    - Android官方强烈建议不要在Android程序里面使用到enum。
- 循环
    - 避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外。
    - 不要在循环当中声明临时变量，不要在循环中捕获异常。
- 对于那些无法避免需要创建对象的情况，可以考虑对象池模型。
    - 通过对象池来解决频繁创建与销毁的问题。
    - 需要注意结束使用之后，需要手动释放对象池中的对象
- 避免大量使用注解、反射 
- BitMap隐患
    - Bitmap的不当处理极可能造成OOM，绝大多数情况应用程序OOM都是因这个原因，在操作的时候必须小心。
    - 及时释放recycle。由于Dalivk并不会主动的去回收，需要开发者在Bitmap不被使用的时候recycle掉。
    - 设置一定的压缩率。需求允许的话，应该去对BItmap进行一定的缩放，通过BitmapFactory.Options的inSampleSize属性进行控制。仅只想获得Bitmap的属性，其实并不需要根据BItmap的像素去分配内存，只需在解析读取Bmp的时候使用BitmapFactory.Options的inJustDecodeBounds属性。
    - 建议在加载网络图片的时候，使用软引用或者弱引用并进行本地缓存，推荐使用android-Volley,Picasso、Fresco
- 避免使用Protocal Buffer可能导致方法数与类的个数增加
- 使用集合对象，如果事先知道其大小，则可以在构造方法中设置初始大小
- 如果对于线程安全没有要求，尽量使用线程不安全的集合对象


---

# 数据存储优化

数据类型选择

- 有较多的字符串拼接并且非并发情况下用StringBuilder代替String
- 如果知道字符串长度，可以直接new StringBuilder(num)指定初始大小，减少空间不够时的再次分配
- 64位类型如long double的处理比32位如int慢
- 常量请使用static final来修饰，final类型存储在常量区中读取效率更高


数据结构选择

- ArrayList和LinkedList的选择，ArrayList根据index取值更快，LinkedList更占内存、随机插入删除更快速、扩容效率更高。一般推荐ArrayList
- ArrayList、HashMap、LinkedHashMap、HashSet的选择，hash系列数据结构查询速度更优，ArrayList存储有序元素，HashMap为键值对数据结构，LinkedHashMap可以记住加入次序的hashMap，HashSet不允许重复元素。
- HashMap、WeakHashMap选择，WeakHashMap中元素可在适当时候被系统垃圾回收器自动回收，所以适合在内存紧张型中使用。
- Collections.synchronizedMap和ConcurrentHashMap的选择，ConcurrentHashMap为细分锁，锁粒度更小，并发性能更优。Collections.synchronizedMap为对象锁，自己添加函数进行锁控制更方便
- 使用特定Map容器容器，取代HashMap来避免autoboxing带来的效率问题.(Don't use HashMap since it's memory inefficient. Putting a single entry into a HashMap requires 32 bytes.）
      
   ![](https://i.imgur.com/FoZtb5Q.jpg)
        
- 特定Sparse容器（SparseArray、SparseBooleanArray、SparseIntArray） 
- 在满足：对象个数的数量级最好是千以内和数据组织形式包含Map结构，考虑使用ArrayMap


# [缓存优化 ](http://vivianking6855.github.io/2018/04/12/Android-cache/)


# [算法](https://blog.csdn.net/vivian_king/article/details/79624965)优化

需要具体问题具体分析，尽量不用O(n*n)时间复杂度以上的算法，必要时候可用空间换时间

- 避免用浮点计算
- 尽量避免试用递归，它非常的耗费memory，虽然它很快
- 查询考虑hash和二分，尽量不用递归。

可以从结构之法 算法之道或微软、Google等面试题学习
 
# JNI

Java需要Dalvik的JIT编译器将Java字节码转换成本地代码运行，而本地代码可以直接由设备管理器直接执行，节省了中间步骤，所以执行速度更快。

不过需要注意从Java空间切换到本地空间需要开销，同时JIT编译器也能生成优化的本地代码，所以糟糕的本地代码不一定性能更优。

TODO

---

# Location

开启定位功能是一个相对来说比较耗电的操作，通常使用类似下面这样的代码来发出定位请求：

   ![](https://i.imgur.com/gLmQzTe.jpg)

setInterval()指的意思是每隔多长的时间获取一次位置更新，时间相隔越短，自然花费的电量就越多，但是时间相隔太长，又无法及时获取到更新的位置信息。

优化点：可以通过判断返回的位置信息是否相同，从而决定设置下次的更新间隔是否增加一倍，通过这种方式可以减少电量的消耗，如下图所示：

![](https://i.imgur.com/ruw3q8k.jpg)

---

# 网络

网络请求的操作是非常耗电的，其中在移动蜂窝网络情况下执行网络数据的请求则尤其比较耗电。

- 关于如何减少移动网络下的网络请求的耗电量，有两个重要的原则需要遵守
    - 第一个是减少移动网络被激活的时间与次数
    - 第二个是压缩传输数据。
- 绝对坚决肯定不应该使用Polling(轮询)的方式去执行网络请求，这样不仅仅会造成严重的电量消耗，还会浪费许多网络流量，例如：

# 其他

1. 必要时设定android:largeHeap，使用 android:largeHeap="true"标记 (API Level >= 11) ，但在一些严格的机器上不work，使用时还是需要android getMemoryClass()来判断
    - 在 AndroidManifest.xml中的 Application节点中声明即可分配到更大的堆内存
    - android:largeHeap标记在 Android系统应用中也有广泛的应用 ,比如 Launcher, 
    - Browser这些内存大户上均有使用
2. 使用优化后的lib和硬件加速。使用外部lib时注意，确保使用的是为移动环境优化后的lib
3. 关注lint工具所提出的建议，进行优化。


# Reference

[更多性能优化相关：性能优化目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)

