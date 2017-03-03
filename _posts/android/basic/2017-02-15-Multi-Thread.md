---
layout: post
title: 2017-2-15 Android的线程和线程池
date: 2017-2-15
excerpt: "Android的线程和线程池"
tags: [Android 基础]
comments: true
---

# Android的线程和线程池

## 一、Android的线程

线程在Android中是一个很重要的概念。Android沿用了Java的线程模型，线程也分为主线程（UI线程）和子线程。

A.	主线程：主要处理和界面相关的事情

通常用于处理/接收用户的输入，将运算的结果反馈给用户。UI界面不能阻塞的，用户无法输入。

这里要谨记：更新UI应该是一个短操作（不应该超过5秒，不然会像未使用Handler一样，UI会报ANR异常）, 在更新操作处理期间，UI会被冻结阻塞甚至程序会崩溃。

B.	子线程：往往用于执行耗时的操作

例如一样网络数据加载，I/O操作，DB操作等

### Android中线程形态

Android中扮演线程角色的有：

- Thread
- AsyncTask
    - 底层用到了线程池。封装了线程池和Handler，主要用于在子线程中更新UI
    - Android 3.0之后是串行的线程池
- IntentService：底层直接使用了线程
    - 所有task执行完毕后，IntentService会自动退出
    - 它是一个服务，可以更方便地执行后台任务，IntentService内部采用HandlerThread来执行任务，当任务执行完毕后IntentService会自动退出。虽然看起来很像一个后台线程，但是IntentService是一种服务。较普通的后台线程它不容易被系统杀死，从而可以尽量保证任务的执行。
    - 适合于高优先级的后台任务。
- HandlerThread ：底层直接使用了线程
    - 具有消息循环的线程，内部可以使用Handler

它们的本质都是传统的线程。

在操作系统中，线程是操作系统调度的最小单元。但是线程是一种受限的系统资源，不可能无限地产生，并且线程的创建和销毁都会有相应的开销。

### 线程实践

1.	AsyncTask 模拟下载文件。注意事项
    - AsyncTask的execute必须在UI线程调用
    - 不要直接调用onPreExecute, doInBackground等方法
    - 一个AsyncTask对象只能执行一次，即只能调用一次execute方法

核心Code

    private class DownloadAsyncTask extends AsyncTask<URL, Integer, Long> { //(Params, Progress, Result)
        protected Long doInBackground(URL... urls) { // run in thread pool
            int count = urls.length;
            long totalSize = 0;
            for (int i = 0; i < count; i++) {
                totalSize += urls[i].toString().length(); // simulate each file size
                publishProgress((int) ((i / (float) count) * 100));
                // Escape early if cancel() is called
                if (isCancelled()) {
                    break;
                }
            }
            return totalSize;
        }

        protected void onProgressUpdate(Integer... progress) { // run in main thread
            mTVShow.append("progress  =  " + progress[0] + "\r\n");
        }

        protected void onPostExecute(Long result) {// run in main thread
            mTVShow.append("doInBackground return totalSize: " + result + "\r\n");
        }
    }

2.	IntentService
    - 适合于高优先级的后台任务。
    - 所有task执行完毕后，IntentService会自动退出
    - 可用Messenger，Broadcast等方式更新UI

核心Code

    public void runIntentService(View v) {
        mTVShow.setText("");
        Intent intent = new Intent(this, UserIntentService.class);
        // task query
        intent.putExtra(Const.KEY_ACTION, Const.ACTION_QUERY);
        intent.putExtra(Const.KEY_MAIN_MESSENGER, new Messenger(mMainHandler));
        startService(intent);
        // task update
        intent.putExtra(Const.KEY_ACTION, Const.ACTION_UPDATE);
        intent.putExtra(Const.KEY_MAIN_MESSENGER, new Messenger(mMainHandler));
        startService(intent);
        // task del
        intent.putExtra(Const.KEY_ACTION, Const.ACTION_DEL);
        intent.putExtra(Const.KEY_MAIN_MESSENGER, new Messenger(mMainHandler));
        startService(intent);
    }

## 二、线程池

在一个进程中频繁地创建和销毁线程，不是高效的做法。正确的做法是采用线程池。一个线程池中会缓存一定数量的线程，通过线程池可以避免因为频繁创建和销毁线程所带来的系统开销。

假设完成一项任务的时间 = T1 创建线程时间 + T2 在线程中执行任务的时间 + T3 销毁线程时间。

当T1 + T3 远大于 T2时，采用多线程技术可以显著减少处理器单元的闲置时间，增加处理器单元的吞吐能力。

### 线程池的优点

1. 重用线程池中的线程，避免因线程的创建和销毁所带来的性能开销。
2. 能有效的控制线程的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象
3. 能够对线程进行简单的管理，并提供定时执行以及制定间隔循环执行等功能

### ThreadPoolExecutor

Android中的线程池概念来源于Java的Executor，Executor是一个接口，真正的线程池的实现为ThreadPoolExecutor。

1.	三个重要参数
    - corePoolSize 

    核心线程数，默认情况下，核心线程会在线程池中一直存活，即使他们处于闲置状态。不过可以通过allowCoreThreadTimeOut属性搭配keepAliveTime参数，实现核心线程超时终止

    - maximumPoolSize

    线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务将会被阻塞
    
    - keepAliveTime

    非核心线程闲置时的超时时长，超过非核心线程 就会被回收。当allowCoreThreadTimeOut为true时，也会作用于核心线程
    
2.	ThreadPoolExecutor执行的大致规则

    - 如果线程池中的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务
    - 如果线程池中的线程数据量已经达到或超过核心线程的数量，那么任务会被插入到任务队列中等待执行
    - 如果2中无法将任务插入到任务队列，往往是由于任务队列已满，这个时候如果线程数量未达到线程池规定的最大值，那么会立刻启动一个非核心线程来执行任务
    - 如果3中线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务。ThreadPoolExcecutor会调用RejectedExecutionHandler来通知调用者

### [AsynTask中的线程池配置](https://android.googlesource.com/platform/frameworks/base/+/android-cts-7.1_r2/core/java/android/os/AsyncTask.java)

- 核心线程数： [2，4]
- 线程池的最大线程数：CPU核心数的2倍 + 1
- 核心线程和非核心线程闲置超时时间为30秒
- 任务队列的容量为128


        private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
            // We want at least 2 threads and at most 4 threads in the core pool,
            // preferring to have 1 less than the CPU count to avoid saturating
            // the CPU with background work
            private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
            private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
            private static final int KEEP_ALIVE_SECONDS = 30;
            private static final ThreadFactory sThreadFactory = new ThreadFactory() {
                private final AtomicInteger mCount = new AtomicInteger(1);
                public Thread newThread(Runnable r) {
                    return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
                }
            };
            private static final BlockingQueue<Runnable> sPoolWorkQueue =
                    new LinkedBlockingQueue<Runnable>(128);
            /**
             * An {@link Executor} that can be used to execute tasks in parallel.
             */
            public static final Executor THREAD_POOL_EXECUTOR;
            static {
                ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                        CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                        sPoolWorkQueue, sThreadFactory);
                threadPoolExecutor.allowCoreThreadTimeOut(true);
                THREAD_POOL_EXECUTOR = threadPoolExecutor;
        }
        
### 四类常见的线程池

Android中最常见的四类具有不同功能特性的线程池。他们都是直接或间接通过配置ThreadPoolExecutor来实现自己的功能特性。

1. FixedThreadPool
    - 线程数量固定的线程池。只有核心线程。
    - 没有超时机制，任务队列也没有大小限制
    - 即使线程处于空闲状态也不会被回收，除非线程池关闭。
    - 当所有线程都处于活动状态时，新任务都会处于等待状态。
    - 适合于需要快速响应外界的情景

            public static ExecutorService newFixedThreadPool(int nThreads) {
                return new ThreadPoolExecutor(nThreads, nThreads,
                                              0L, TimeUnit.MILLISECONDS,
                                              new LinkedBlockingQueue<Runnable>());
            }

2. CachedThreadPool
    - 线程数量不定的线程池，只有非核心线程
    - 最大线程数为Integer.MAX_VALUE，相当于任意大。
    - 队列使用SynchronousQueue，可理解为无法存储元素的队列，相当于空集合，线程池的任务都会立即被执行
    - 空闲线程超时时长60秒，空闲时间超出就会被回收。
    - 当所有线程都处于活动状态时，会创建新的线程来处理任务。
    - 适合于大量的耗时较少的任务。

            public static ExecutorService newCachedThreadPool() {
                return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                              60L, TimeUnit.SECONDS,
                                              new SynchronousQueue<Runnable>());
            }

3. ScheduledThreadPool
    - 核心线程数量固定，非核心线程数量没有限制。
    - 当非核心线程空闲会被立即回收。
    - 适用于定时任务和具有固定周期的重复任务。

            private static final long DEFAULT_KEEPALIVE_MILLIS = 10L;
            public ScheduledThreadPoolExecutor(int corePoolSize) {
                super(corePoolSize, Integer.MAX_VALUE,
                      DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                      new DelayedWorkQueue());
            }

4. SingleThreadExecutor
    - 只有一个核心线程，确保所有的任务都在一个线程中按顺序执行。
    - 意义在于统一所有的外界任务到一个线程中，任务之间不需要处理线程同步的问题。

            public static ExecutorService newSingleThreadExecutor() {
                return new FinalizableDelegatedExecutorService
                    (new ThreadPoolExecutor(1, 1,
                                            0L, TimeUnit.MILLISECONDS,
                                            new LinkedBlockingQueue<Runnable>()));
            }

### 线程池实践

    private void initThreadPool() {
        fixedThreadPool = Executors.newFixedThreadPool(4);
        cachedThreadPool = Executors.newCachedThreadPool();
        scheduledThreadPool = Executors.newScheduledThreadPool(4);
        singleThreadExecutor = Executors.newSingleThreadExecutor();
    }

    public void runThreadPool(View v) {
        fixedThreadPool.execute(command);

        cachedThreadPool.execute(command);

        // 2000ms后执行command
        scheduledThreadPool.schedule(command, 2000, TimeUnit.MILLISECONDS);
        // 延迟10ms后，每隔1000ms执行一次command
        scheduledThreadPool.scheduleAtFixedRate(command, 10, 1000, TimeUnit.MILLISECONDS);

        singleThreadExecutor.execute(command);
    }


[github Code](https://github.com/vivianking6855/android-advanced)



<br/>
<br/>


> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)

> [聊聊并发](http://ifeve.com/volatile/) 
