---
layout: post
title: 2017-2-16 线程同步
date: 2017-2-16
excerpt: "线程同步"
tags: [Android]
comments: true
---

## 基础概念

线程的“同步”不是指平常所说的两件事情同时进行，是使多个线程之间协调工作。

而且常常是避免两个线程同时进行某些操作。比如同时访问同一个共享资源。

可以理解为同步是：因逻辑需要或者会发生冲突，需要暂停某个线程，然后等待合适的时机恢复。

下面介绍同步相关知识点：

1.	最基础的Synchronized关键字
2.	多线程简化开发库 java.util.concurrent

## Synchronized关键字

- synchronized 是Java提供的用于实现同步的关键字
- synchronized关键字用来取得一个对象的同步锁
- synchronized 可以用来修饰的方法，代码块和对象/类

### 同步锁的原理

Java中每个对象都仅有一个内置同步锁。有线程持有锁时，其他任何线程都不能访问被锁住的部分，直到该线程释放同步锁

### synchronized的使用

1.	取得对象的同步锁 ：synchronized(object)
2.	Synchronized修饰方法（同步方法）：
    - synchronized void show() 相当于synchronized(this)的缩写
    - static synchronized void show() 相当于  synchronized(当前类名.class) 

### synchronized之协调和调度

线程的同步锁对象可以调用notify、notifyAll、wait 这三个方法来实现线程间协调和调度

- wait() 
    - 导致当前线程等待，直到其他线程调用同步锁对象notify()方法或notifyAll()方法来唤醒该线程。
    - 调用wait()方法的当前线程会释放对该同步监视器的锁定。
- notify()
    - 唤醒在此同步锁上等待的单个线程。如果所有线程都在此同步监视器上等待，则会选择任意一个线程唤醒。
    - 但是只有当前线程放弃此同步锁后，才可以执行被唤醒的线程。
- notifyAll()
    - 唤醒在此同步锁上等待的所有线程。同样只有当前线程放弃此同步锁后，才可以执行被唤醒的线程。

### synchronized实践

1.实现数字的正确输出
  
  - 输出超出期望的原因：同步锁的不一致性。修改方法可参考
      - 使用静态变量作为同步锁
      - 使用类作为同步锁

核心Code

     public class SyncBasic {
        private final static String TAG = SyncBasic.class.getSimpleName();
    
        private int countFlag = 1;
        private static Integer sLocker = 1;
        private Handler mUIHandler = null;
    
        public SyncBasic(Handler handler) {
            mUIHandler = handler;
        }
    
        public void beginTest() {
            Runnable operate = new Runnable() {
                @Override
                public void run() {
                    new Operation().operate();
                }
            };
            new Thread(operate,"Thread-01").start();
            new Thread(operate,"Thread-02").start();
        }
    
        private class Operation {
            public void operate() {
                //synchronized (SyncBasic.sLocker) {
                //synchronized (Operation.class) {
                synchronized (this) {
                    countFlag++;
                    try {
                        Thread.sleep(new Random().nextInt(5));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    countFlag--;
    
                    String show = "Thread: " + Thread.currentThread().getName()
                            + " /countFlag : " + countFlag;
    
                    Log.d(TAG, show);
                    notifyUI(show + "\r\n");
                }
            }
        }
    
        private void notifyUI(String show) {
            Message msg = mUIHandler.obtainMessage(Const.MSG_TO_UI);
            msg.obj = show;
            msg.sendToTarget();
        }
    
    }

2.实现期望的逻辑输出 : wait(), notify()的使用

期望输出： SKY, CLOUD, SUN, RAIN

核心Code:

    public class SyncAdvanced {
        private final static String TAG = SyncAdvanced.class.getSimpleName();
    
        private final static String SKY = "\r\n Sky";
        private final static String RAIN = "\r\n Rain";
        private final static String CLOUD = "\r\n Cloud";
        private final static String SUN = "\r\n Sun";
        private Integer mLocker = 1;
    
        public void beginTest() {
            // new thread
            new Thread(new GameRunnable()).start();
    
            synchronized (mLocker) {
                try {
                    Log.d(TAG, SKY);
                    // wait, release locker
                    mLocker.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
    
                Log.d(TAG, RAIN);
            }
        }
    
        private class GameRunnable implements Runnable {
            public void run() {
                SystemClock.sleep(100);
    
                synchronized (mLocker) {
                    Log.d(TAG, CLOUD);
                    // notify
                    mLocker.notify();
                    Log.d(TAG, SUN);
                }
            }
        }
    
    }

## concurrent包

虽然synchronized已经足够强大，但想要用好也挺不容易。

从JDK 1.5开始，增加了java.util.concurrent包，它的引入大大简化了多线程程序的开发。

内容涵盖了并发集合类、线程池机制、同步互斥机制、线程安全的变量更新工具类、锁等常用工具。提供了很多实用并发程序模型

- Executor 创建各类线程池：newFixedThreadPool、newCachedThreadPool等
- ExecutorService  线程池管理。execute方法可以把Runnable,Callable提交到线程池中。
- BlockingQueue ：阻塞队列。
    - 阻塞队列的概念是一个指定长度的队列：如果队列满了，添加新元素的操作会被阻塞等待，直到有空位为止。
    - 当队列为空时候，请求队列元素的操作同样会阻塞等待，直到有可用元素为止。
    - 多用于多线程的排队等候，特别是生产者-消费者的情景
- Future 与Runnable,Callable进行交互的接口。线程执行结束后取返回的结果等等，还提供了cancel终止线程。
- CompletionService ExecutorService的扩展，可以获得线程执行结果的
- Semaphore 一个计数信号量
- ReentrantLock 可重入的互斥锁定 Lock，功能类似synchronized，但要强大的多
- CountDownLatch 在完成其他线程中操作之前，允许一个或多个线程一直等待
- CyclicBarrier 允许一组线程互相等待，直到到达某个公共屏障点
- [CopyOnWriteArrayList](http://ifeve.com/java-copy-on-write/) 
    - CopyOnWrite容器即写时复制的容器。
    - 通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。
    - 这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。
    - 所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。
- ConcurrentHashMap

部分实践Code




## 实践 生产者-消费者模型

Concurrent经典的例子：生产者和消费者的问题。

1.	生产者不断的生产馒头，放入一个篮子里，而消费者不断的从篮子里拿馒头吃。

2.	当篮子满的时候，生产者自己等待不在生产馒头。

使用current包的BlockingQueue,核心Code:

    //ConsumerRunnable.java
    
    public class ConsumerRunnable implements Runnable {
        private final static String TAG = ConsumerRunnable.class.getSimpleName();
    
        private String name;
        private BlockingQueue<Product> mQueue;
    
        public ConsumerRunnable(String name, BlockingQueue<Product> queue) {
            this.name = name;
            mQueue = queue;
        }
    
        @Override
        public void run() {
            for (int i = 0; i < Const.ALL_NUM; i++) {
                // consume product
                try {
                    Product item = mQueue.take();
                    Log.d(TAG, name + " : " + item.toString());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
    
                SystemClock.sleep(Const.TIME_CONSUME);
            }
        }
    }
    
    // MakerRunnable.java
    public class MakerRunnable implements Runnable {
        private final static String TAG = MakerRunnable.class.getSimpleName();
    
        private String name;
        private BlockingQueue<Product> mQueue;
    
        public MakerRunnable(String name, BlockingQueue<Product> queue) {
            this.name = name;
            mQueue = queue;
        }
    
        @Override
        public void run() {
            for (int i = 0; i < Const.ALL_NUM; i++) {
                Product item = new Product(i, "name " + i);
                try {
                    mQueue.put(item);
                    Log.d(TAG, name + " : " + item.toString());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                SystemClock.sleep(Const.TIME_MAKE);
            }
        }
    }
    
    //ProductManager.java
    public class ProductManager {
        private final static String TAG = ProductManager.class.getSimpleName();
    
        // Queue
        private final BlockingQueue<Product> mQueue = new LinkedBlockingDeque<Product>(2);
    
        public void beginTest() {
            //ExecutorService exec = Executors.newFixedThreadPool(3);
            ExecutorService exec = Executors.newCachedThreadPool();
            exec.execute(new ConsumerRunnable("c1", mQueue));
            exec.execute(new ConsumerRunnable("c2", mQueue));
            exec.execute(new MakerRunnable("maker", mQueue));
        }
    
    }


[github Code](https://github.com/vivianking6855/android-advanced)



<br/>
<br/>


> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)

> [聊聊并发](http://ifeve.com/volatile/) 
