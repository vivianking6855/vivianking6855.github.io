---
layout: post
title: ReactNative
date: 2016-07-15
excerpt: "Android性能优化 （一）"
tags: [Android]
comments: true
---

## 内存优化篇

### 内存示意图

![](http://i.imgur.com/2ylcYWn.png)

1. Stack和Heap：

   - Stack空间（进栈和出栈）由操作系统控制，其中主要存储函数地址、函数参数、局部变量、还有一些基础类型等等，所以Stack空间不需要很大，一般为几MB大小。

   - Heap空间的使用由程序员控制，程序员可以使用malloc、new、free、delete等函数调用来操作这片地址空间。

        Heap为程序完成各种复杂任务提供内存空间，所以空间比较大，一般为几百MB到几GB。  
        Heap空间由程序员管理，所以容易出现使用不当导致严重问题。

2. Android中的进程

   - 进程分为native进程（c/c++实现）和java进程（dalvik实例的linux进程。入口函数是java的main函数。
    
   - 每一个android上的java进程实际上就是一个linux进程只是进程中多了一个dalvik虚拟机实例。
    
   - Android系统中的应用程序基本都是java进程，如桌面、电话、联系人、状态栏等等。

3. Android中进程的堆内存 Heap

  - 进程空间中的heap空间是我们需要重点关注的。

  - heap空间完全由程序员控制，我们使用的malloc、C++ new和java new所申请的空间都是heap空间。

  - C/C++申请的内存空间在native heap中，而java申请的内存空间则在dalvik heap中。

4. Android的 java程序为什么容易出现OOM

   - 这个是因为Android系统对dalvik的vm heapsize作了硬性限制，当java进程申请的java空间超过阈值时，就会抛出OOM异常。
    
        （这个阈值可以是48M、24M、16M等，视机型而定），可以通过adb shell getprop | grep dalvik.vm.heapgrowthlimit查看此值。
    
    - 程序发生OMM并不表示RAM不足，而是因为程序申请的java heap对象超过了dalvik vm heapgrowthlimit。
    
        也就是说，在RAM充足的情况下，也可能发生OOM。


5. 内存泄露

    首先看下dalvik的Garbage Collection，GC会选择回收没有直接或者间接引用到GC Roots的点。如下图蓝色部分。
    
    Java内存泄漏指的是进程中某些对象（垃圾对象）已经没有使用价值了，但是它们却可以直接或间接地引用到gc roots导致无法被GC回收。
    
    无用的对象占据着内存空间，使得实际可使用内存变小，形象地说法就是内存泄漏了。


![](http://i.imgur.com/RuracHg.png)

5.1. 常见的内存泄漏

（1）Context泄漏
    
   示例： 非静态内部类的静态实例容易造成内存泄漏。

   下面的静态内部类对象持有MainActivity的引用，在Activity销毁时MainActivity无法回收。
    
   对于lauchMode不是singleInstance的Activity，应该避免在activity里面实例化其非静态内部类的静态实例

    public class MainActivity extends Activity
    {
       static Demo sInstance = null;
            
        @Override
        public void onCreate(BundlesavedInstanceState)
        {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            if (sInstance == null)
            {
               sInstance= new Demo();
            }
        }
        class Demo
        {
            voiddoSomething()
            {  
                System.out.print("dosth.");
            }
        }
     }
     
以上例子的内存泄漏都是因为Activity的引用的生命周期超越了activity对象的生命周期。也就是常说的Context泄漏，因为activity就是context。

想要避免context相关的内存泄漏，需要注意以下几点：

- 不要对activity的context长期引用(一个activity的引用的生存周期应该和activity的生命周期相同)
- 如果可以的话，尽量使用关于application的context来替代和activity相关的context
- 如果一个acitivity的非静态内部类的生命周期不受控制，那么避免使用它；正确的方法是使用一个静态的内部类，并且对它的外部类有一WeakReference（例如 static Handler创建时可以把外部类的weakreference传进来）


（2）使用handler时的内存问题


   Handler通过发送Message与其他线程交互，Message发出之后是存储在目标线程的MessageQueue中的，而有时候Message也不是马上就被处理的，可能会驻留比较久的时间。
    
   在Message类中存在一个成员变量 target，它强引用了handler实例，如果Message在Queue中一直存在，就会导致handler实例无法被回收，如果handler对应的类是非静态内部类 ，则会导致外部类实例（Activity或者Service）不会被回收，这就造成了外部类实例的泄露。 
    
   所以正确处理Handler等之类的内部类，应该将自己的Handler定义为静态内部类，并且在类中增加一个成员变量，用来弱引用外部类实例，如下：

    
    public class OutterClass  
    {  
            ......  
            ......  
            static class InnerClass  
            {  
                private final WeakReference<OutterClass> mOutterClassInstance;  
                ......  
                ......  
            }  
    }  
    
    
（3）注册某个对象后未反注册 ： 注册广播接收器、注册观察者等等，比如：

   示例：监听系统中的电话服务以获取一些信息(如信号强度等)
    
    假设我们希望在锁屏界面(LockScreen)中，监听系统中的电话服务以获取一些信息(如信号强度等)，则可以在LockScreen中定义一个PhoneStateListener的对象，同时将它注册到TelephonyManager服务中。
    
    对于LockScreen对象，当需要显示锁屏界面的时候就会创建一个LockScreen对象，而当锁屏界面消失的时候LockScreen对象就会被释放掉。
    
    但是如果在释放LockScreen对象的时候忘记取消我们之前注册的PhoneStateListener对象，则会导致LockScreen无法被GC回收。
    
    如果不断的使锁屏界面显示和消失，则最终会由于大量的LockScreen对象没有办法被回收而引起OutOfMemory,使得system_process进程挂掉。

（4）集合中对象没清理造成的内存泄露

　　例如对象的引用的集合不再使用时，如果没有把它的引用从集合中清理掉，这样这个集合就会越来越大。如果这个集合是static的话，那情况就更严重了。

   示例：某公司的ROM的锁屏曾经就存在内存泄漏问题：

    这个泄漏是因为LockScreen每次显示时会注册几个callback，它们保存在KeyguardUpdateMonitor的ArrayList<InfoCallback>、ArrayList<SimStateCallback>等ArrayList实例中。
    
    但是在LockScreen解锁后，这些callback没有被remove掉，导致ArrayList不断增大， callback对象不断增多。
    
    这些callback对象的size并不大，heap增长比较缓慢，需要长时间地使用手机才能出现OOM。
    
    由于锁屏是驻留在system_server进程里，所以导致结果是手机重启。

（5）资源对象没关闭造成的内存泄露，比如(Cursor，File文件等)

   对于资源性对象在不使用的时候，应该立即调用它的close()函数，将其关闭掉，然后再置为null.

（6）构造Adapter时，没有使用缓存的 convertView，可以使用Android最新组件RecyclerView,替代ListView来避免

 (7) Bitmap使用不当
 
 必要的措施：
 
   第一、及时的销毁。

   虽然，系统能够确认Bitmap分配的内存最终会被销毁，但是由于它占用的内存过多，所以很可能会超过Java堆的限制。
    
   因此，在用完Bitmap时，要及时的recycle掉。recycle并不能确定立即就会将Bitmap释放掉，但是会给虚拟机一个暗示：“该图片可以释放了”。


   第二、设置一定的采样率。

   有时候，我们要显示的区域很小，没有必要将整个图片都加载出来，而只需要记载一个缩小过的图片。
   
   这时候可以设置一定的采样率，那么就可以大大减小占用的内存。
   
   如下面的代码：

    private ImageView preview;    
    BitmapFactory.Options options = newBitmapFactory.Options();    
    options.inSampleSize = 2;//图片宽高都为原来的二分之一，即图片为原来的四分之一    
    Bitmap bitmap =BitmapFactory.decodeStream(cr.openInputStream(uri), null, options); preview.setImageBitmap(bitmap);   

   第三、巧妙的运用软引用（SoftRefrence）

   有些时候，我们使用Bitmap后没有保留对它的引用，因此就无法调用Recycle函数。
   
   这时候巧妙的运用软引用，可以使Bitmap在内存快不足时得到有效的释放。如下：

    SoftReference<Bitmap>  bitmap_ref  = new SoftReference<Bitmap>(BitmapFactory.decodeStream(inputstream));   
    ……  
    ……  
    if (bitmap_ref .get() != null)  
              bitmap_ref.get().recycle(); 
              
   PS: 对象的引用强度 SoftRefrence > WeakReference



## 内存检测工具
[leak-canary git hub](https://github.com/square/leakcanary)


> [Android最佳性能实践(一)——合理管理内存 ](http://blog.csdn.net/guolin_blog/article/details/42238627)

