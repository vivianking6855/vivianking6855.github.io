---
layout: post
title: Android性能优化（一）Android的性能优化方法概述
date: 2017-2-27
excerpt: "Android的性能优化方法概述"
categories: Android
tags: [Android 进阶]
comments: true
---

* content
{:toc}


# 一、 布局优化

1. 布局优化的思想：尽量减少布局文件的层级。层级少了，Android绘制的工作量少了，程序的性能自然就高了。

2. 如何进行布局优化

    (1) 删除布局中无用的控件和层级 <br>
    (2）有选择地使用性能较低的ViewGroup
   
   - 比如不嵌套的情况下，用LinearLayout和FrameLayout代替RelativeLayout.
   - RelativeLayout功能比较复杂，布局过程需要花费更多的CPU时间
   - 但是如果要LinearLayout嵌套来代替RelativeLayout，还是建议用RelativeLayout。因为嵌套同样会降低程序的性能  

(3) 采用<include>、<merge>标签和ViewStub

   - <include>主要用于布局重用
   
    例如：
   
       <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
            
            <include layout="@layout/constraint_around" />
        
        </LinearLayout>
   
   - <merge>一般和<include>配合使用，可以降低减少布局的层级

        例如：如果当前布局是竖直方向的LinearLayout，而且被包含的布局文件也是竖直方向的LinearLayout。
        
        那么被包含的布局就可以通过<merge>去掉移除LinearLayout
        
        <merge xmlns:android="http://schemas.android.com/apk/res/android">
        
            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/app_name" />
        
            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/app_name" />
        
        </merge>
   
   - ViewStub 按需加载，当需要时才会将ViewStub中的布局加载到内存
    - 可以通过设定setVisibility(View.VISIBLE)或inflate显示对应的layout
    
    例如 网络 offline or page invalidate
    
        <ViewStub
        android:id="@+id/stub_offline"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inflatedId="@+id/panel_offline"
        android:layout="@layout/constraint_around" />
    

# 二、 绘制优化

绘制优化是指View的onDraw方法要避免执行大量的操作

- onDraw中不要创建新的局部对象。频繁调用时，如果一瞬间产生大量的临时对象会占用过多的内存而且会导致系统更加频发的gc，降低程序的执行效率。
- onDraw中不要做耗时任务，也不能执行成千上万的循环操作，尽管每次循环都很轻量级，但是大量的循环仍然十分强占CPU的时间片，会造成View的绘制过程不流畅。
    - Google官方给出的性能优化典范中的标准，View的绘制频率保证60fps是最佳，要求每帧的绘制时间不超过16ms(16ms = 1000/60)
    - 虽然程序很难保证16ms这个时间，但是尽量降低onDraw方法的复杂度总是有效的。

# 三、 内存泄漏优化

内存泄漏是在开发过程需要重视的问题。内存泄漏优化分两个方面：

- 开发过程中避免写出有内存泄漏的代码
- 通过一些分析工具如Android Studio自动Android Monitor，[leakcanary](https://github.com/square/leakcanary)，FindBugs等

详见：[ Android性能优化 （二）内存 OOM](http://vivianking6855.github.io/Android-optimization-2-OOM/)

# 四、 响应速度优化

响应速度优化核心思想是避免在主线程中做耗时的操作。耗时的操作应该放到异步任务中执行。

# 五、 ListView优化

构造Adapter时，没有使用缓存的 convertView，可以使用Android最新组件RecyclerView,替代ListView来避免

# 六、 Bitmap优化

主要是BitmapFactory.Options 和缓存。详见[ Android Bitmap的加载和Cache](http://vivianking6855.github.io/Android-Bitmap-Cache/)

# 七、 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread. 控制最大并发数等。详见下面两篇blog~

[Android的线程和线程池](http://vivianking6855.github.io/Multi-Thread/)

[线程同步](http://vivianking6855.github.io/Thread-Sync/)

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

- 避免创建过多的对象
- 不要过多使用枚举，枚举占用内容空间比整型大
- 常量请使用static final来修饰
- 使用一些Android特有的数据结构，比如SparseArray，Pair等，它们都具有更好的性能
- 适当使用软引用和弱引用
- 采用内存缓存和磁盘缓存
- 尽量采用静态内部类，避免潜在的由于内部类而导致的内存泄漏。
- 避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外
- 自定义View中避免在onDraw方法里面执行复杂的操作，避免创建对象。
- 对于那些无法避免需要创建对象的情况，可以考虑对象池模型。
    - 通过对象池来解决频繁创建与销毁的问题。
    - 需要注意结束使用之后，需要手动释放对象池中的对象。


<br/><br/>


> 《Android开发艺术探索》