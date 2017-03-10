---
layout: post
title: IPC - Inter-Process Communication 进程间通信
date: 2017-1-23
excerpt: "IPC 进程间通信"
categories: Android
tags: [Android 基础]
comments: true
lefttrees: true
---

* content
{:toc}



# 一、IPC含义

IPC Inter-Process Communication.含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。


# 二、Android中的多进程模式

在Android中使用多进程只有一种方法：给四大组件（Activity，Service，Receiver,ContentProvider)在AndroidMenifest中制定android:process属性

使用多进程会有下面几个方面的问题：

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharePreference的可靠性下降（SharePreference不支持两个进程同时去执行写操作）
- Application会多次创建


# 三、IPC的基础概念

- 当通过Intent和和Binder传输数据时，需要用到Serializable和Parcelable把数据序列化
- Binder
- Binder实现了IBinder接口
    - 从IPC角度来说Binder是跨进程通信的一种方式
    - 从framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManger等）和形影ManagerService的桥梁
    - 从Android应用层来说，Binder是客户端和服务器端进行通信的媒介
    - Binder主要用在Service中，包括AIDL和Messenger（底层其实是AIDL)

# 四、Android中的IPC方式

跨进程通信方式有很多：Intent传递Bundle数据，共享文件，Binder，Intent传递Bundle数据,ContentProvider,Socket.

下面一一来看下每个方式。

## 1. Intent附加extras传递信息

三大组件（Activity，Service，Receiver)都支持Intent传递Bundle数据（Bundle有实现Parcelable接口）

Intent 方式较为简单，暂不包含在示例中

## 2. 通过共享文件的方式共享数据

- file文件
    - android基于Linux对文件并发读写没有限制
    - 适用于对数据同步要求不高的进程之间的通信
    - 需要处理文件的并发读写问题，特别是并发写（尽量避免或多线程同步来限制多个进程写操作）
- SharedPreferences(不建议用在跨进程间通讯）
    - 目录位于data/data/package name/shared_prefs目录下
    - 本质上也是文件
    - 系统对它的读写有一定的缓存策略，即内存中会有一份缓存，多进模式下读写不可靠

file文件在并发读写时会有问题，而SharedPreferences多进模式下读写不可靠，暂不包含在示例中

## 3. Messenger - 轻量级的IPC方案

Messenger：信使，可以在不同进程中传递Message对象。是一种轻量级的IPC方案。底层实现也是AIDL，不过系统做了封装。

- Messenger的底层其实就是AIDL，它对AIDL做了封装，可以更简洁的进行进程间通信
- 以串行方式，一个一个处理客户端发来的消息。不会有线程同步的问题
- 不适合于有大量并发请求的场景

实现Messenger的步骤：

1）服务器端进程

- 创建Service来处理客户端的连接请求
- 创建Handler并通过它创建一个Messenger对象
- Service的onBind中返回Messenger对底层的Binder

Service放在独立的process运行

        <service
            android:name=".messenger.MessengerService"
            android:enabled="true"
            android:exported="true"
            android:process=":remote">
            <intent-filter>
                <action android:name="com.vv.ipc.messenger.MessengerService.launch" />
            </intent-filter>
        </service>

核心Code:

    public class MessengerService extends Service {
        private static final String TAG = MessengerService.class.getSimpleName();
    
        // messenger used to return mMessenger.getBinder when bind Service
        private final Messenger mMessenger = new Messenger(new MessengerHandler());
    
        // handler to deal with msg from client
        private static class MessengerHandler extends Handler {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case Const.MSG_FROM_CLIENT:
                        dealWithClientMsg(msg);
                        break;
                    default:
                        super.handleMessage(msg);
                }
            }
        }
    
        public MessengerService() {
        }
    
        private static void dealWithClientMsg(Message msg) {
            Log.d(TAG, "get msg :" + msg.getData().getString(Const.KEY_FROM_CLIENT));
            SystemClock.sleep(2000);
            Messenger client = msg.replyTo;
            Message reply = Message.obtain(null, Const.MSG_FROM_SERVICE);
            Bundle bundle = new Bundle();
            bundle.putString(Const.KEY_FROM_SERVICE, "service reply at " + Const.DATA_FORMAT.format(new Date()));
            reply.setData(bundle);
            try {
                client.send(reply);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    
        @Override
        public IBinder onBind(Intent intent) {
            return mMessenger.getBinder();
        }
    }

2）客户端进程

- bindService
- bind成功后拿到IBinder对象，创建一个Messenger
- 通过Messenger就可以像服务器发送消息，消息类型为Message对象。
- 如果服务器端要回应客户端，就和服务器端一样，传给服务器端一个Messenger
    - 创一个Handler并创建一个新的Messenger
    - 把Messenger通过Message的replayTo参数传给服务端。
    - 服务端通过replyTo参数（Messenger）回应客户端。

核心Code:
    
    public class MessengerActivity extends AppCompatActivity {
        private static final String TAG = MessengerActivity.class.getSimpleName();
        private static final String SERVICE_URI = "com.vv.ipc.messenger.MessengerService.launch";
    
        private TextView mTVShow;
    
        // service and reply messenger
        private Messenger mService;
        private Messenger mGetReplyMessenger = new Messenger(new MessengerHandler());
    
        // handler to update ui
        private class MessengerHandler extends Handler {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case Const.MSG_FROM_SERVICE:
                        mTVShow.append(msg.getData().getString(Const.KEY_FROM_SERVICE) + "\r\n");
                        break;
                    default:
                        super.handleMessage(msg);
                }
            }
        }
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_messenger);
    
            initView();
    
            // bind service : after 5.0 must follow
            // https://developer.android.com/google/play/billing/billing_integrate.html#billing-requests
            Intent intent = new Intent(SERVICE_URI);
            intent.setPackage(getPackageName());
            bindService(intent, mServiceConn, Context.BIND_AUTO_CREATE);
        }
    
        private void initView() {
            mTVShow = (TextView) findViewById(R.id.show);
            mTVShow.setMovementMethod(ScrollingMovementMethod.getInstance());
        }
    
        private ServiceConnection mServiceConn = new ServiceConnection() {
            public void onServiceConnected(ComponentName className, IBinder service) {
                mService = new Messenger(service);
                Log.d(TAG, "bind service");
            }
    
            public void onServiceDisconnected(ComponentName className) {
            }
        };
    
        public void sendMsgToService(View v) {
            if(mService == null){
                Log.w(TAG, "mService == null");
                return;
            }
    
            Message msg = Message.obtain(null, Const.MSG_FROM_CLIENT);
            Bundle data = new Bundle();
            data.putString(Const.KEY_FROM_CLIENT, "hello, I' client." + Const.DATA_FORMAT.format(new Date()));
            msg.setData(data);
            msg.replyTo = mGetReplyMessenger;
            try {
                mService.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    
        @Override
        public void onDestroy() {
            super.onDestroy();
            if (mService != null) {
                unbindService(mServiceConn);
            }
        }
    }


Messenger是轻量级的进程间通信，以串行方式处理客户端发来的消息。不适合于有大量并发请求的场景。

主要为了传递消息，如果要跨进程调用服务器端的方法，可以使用AIDL来实现跨进程的方法调用。

## 4. AIDL - 最常用的进程间通讯方式

Messenger是串行的方式处理消息，如果大量的消息同时发送的情景就不太合适了。但是我们可以使用AIDL来实现跨进程的方法调用。

AIDL是最常用的进程间通讯方式，是日常开发中涉及进程间通信是的首选。

AIDL进行进程间通讯的流程

1）服务器端进程

- 创建一个Service用来监听客户端的连接请求
- 创建AIDL文件，将透给客户端的接口在AIDL中声明
- Service中实现这个AIDL接口

服务器端功能说明：

- 获取书单
- 添加书
- 注册新书通知
- 服务器端的方法本身就运行在服务器端的Binder线程池中，所以服务器端方法本身就可以执行大量耗时操作。
    - 客户端可以在非UI线程中调用
    - 切记不要在服务器方法中开线程去进行异步任务，除非你明确知道自己在干什么。
    - 例如 getBookList 接口
- 远程服务器端需要调用客户端的listener中的方法时，被调用的方法运行在客户端的Binder线程池中。
    - 确保调用是在服务器端非UI线程中
    - 可以在服务器端中调用客户端的耗时方法。负责服务器无法响应
    - 例如： 服务器端的 listener.onNewBookArrived
- Linsener接口的unregister需要用到RemoteCallbackList<INewBookListener>
    - 通过Binder传递到服务器端后，会产生两个全新的对象。对象是不能跨进程传输的，对象的跨进程传输本质上都是反序列化的过程。
    - RemoteCallbackList是系统专门提供的用于删除跨进程listener的接口。
    - RemoteCallbackList无法像List一样操作。遍历需要beginBroadcast和finishBroadcast配对使用。哪怕是仅获取元素个数。

            final int N = mListenerList.beginBroadcast();
            mListenerList.finishBroadcast();
            Log.d(TAG, "registerListener, current size:" + N);
- 为了程序的健壮性，服务器进程意外停止需要重新连接服务。Binder是可能意外死亡的。
    - 方案一：设定DeathRecipient. 它运行在Client的Binder线程池，不能访问UI

            private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
                @Override
                public void binderDied() {
                    Log.d(TAG, "Binder died, tid = " + Thread.currentThread().getName());
                    if (mRemoteBookManager == null) {
                        return;
                    }
                    mRemoteBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
                    mRemoteBookManager = null;
        
                    // release resource or rebind server here
                }
            };
    
    - 方案二：onServiceDisconnected中重新连接远程服务器。 运行在UI线程，可以访问UI

            public void onServiceDisconnected(ComponentName className) {
                mRemoteBookManager = null;
                Log.d(TAG, "onServiceDisconnected, tid = " + Thread.currentThread().getName());
            }

- AIDL中使用权限验证
- 默认情况下远程Service是任何人都可以连接。必须给服务器加入权限验证功能。
- 方案一：onBind中进行验证。验证方式有很多，例如使用Permission验证方式。Messenger通讯也可以使用此方案

        <permission
            android:name="com.vv.ipc.permission.ACCESS_BOOK_SERVICE"
            android:protectionLevel="normal" />
    
        <uses-permission android:name="com.vv.ipc.permission.ACCESS_BOOK_SERVICE" />
    
        @Override
        public IBinder onBind(Intent intent) {
            int check = checkCallingOrSelfPermission(Const.PERMISSION_ACCESS_BOOK_SERVICE);
            Log.d(TAG, "onBind check = " + check);
            if (check == PackageManager.PERMISSION_DENIED) {
                Log.d(TAG, "onBind permission deny");
                return null;
            }
            return mBinder;
        }
  
- 方案二：onTransact中进行验证。验证方式可以使用上面的Permission，也可以拿到Client端的Uid，Pid，用package name等来验证  


    
核心Code

- [BookManagerService.java](https://github.com/vivianking6855/android-advanced/blob/master/IPC/app/src/main/java/com/vv/ipc/book/BookManagerService.java)

2）客户端

- 绑定Service
- bind成功后将Binder对象转成AIDL接口类型
- 调用AIDL接口的方法

客户端功能说明：

- 获取书单
- 添加书
- 注册新书通知
 
核心Code: [BookManagerActivity.java](https://github.com/vivianking6855/android-advanced/blob/master/IPC/app/src/main/java/com/vv/ipc/book/BookManagerActivity.java) 


3）AIDL接口创建 和 AIDL接口的实现（远程服务器端Service的实现）

- 支援的数据类型：java基本类型,String,List(ArrayList),Map(HashMap),Parcelable,AIDL接口
- 自定义Parcelable对象和AIDL对象必须要显式import
- 自定义Parcelable对象，必须新建一个和它同名的AIDL文件，并在其中声明Parcelable类型。这两个文件要在同一个package下面
- AIDL除了基本数据类型，其他类型的参数必须标上方向：in,out, inout
- AIDL接口只支持方法，不支持声明静态变量。

核心Code

    // Book.aidl
    package com.vv.ipc.book;
    
    parcelable Book;
    
    // IBookManager.aidl
    package com.vv.ipc;
    
    import com.vv.ipc.book.Book;
    import com.vv.ipc.INewBookListener;
    
    interface IBookManager {
        // get all book list
        List<Book> getBookList();
        // add one book
        void addBook(in Book book);
        // get book list size
        int getBookSize();
        // register and unregister new book listener
        void registerListener(INewBookListener listener);
        void unregisterListener(INewBookListener listener);
    }

    
    // INewBookListener.aidl
    package com.vv.ipc;
    
    import com.vv.ipc.book.Book;
    
    interface INewBookListener {
        void onNewBookArrived(in Book newBook);
    }


[github Code](https://github.com/vivianking6855/android-advanced)

## 5. ContentProvider 

ContentProvider天生支持跨进程数据传递，是Android中提供的专门用于不同应用间进行数据共享的方式。

和Messenger一样，底层实现同样也是Binder.但是使用过程比AIDL简单许多，系统有做封装。

[Android之ContentProvider详解](http://blog.csdn.net/x605940745/article/details/16118939)

## 6. Socket

Socket网络通讯也可以实现数据传递

# 五、Binder连接池

大量的业务模块都需要使用AIDL来进行进程通讯的时候，不可能每个业务模块创建一个Service。

我们要减少Service的数量，将所有的AIDL放在同一个Service中去管理。

Binder连接池的工作机制：

- 每个业务模块创建自己的AIDL接口并实现。
- 每个业务模块之间不能有耦合
- 单独实现细节后，向服务器提供自己的唯一标识和其对应的Binder对象
- 服务器端只需要一个Service，服务器提供一个queryBinder接口，依据业务模块的特征来返回相应的Binder对象
- 不同业务模块拿到所需Binder后，可以进行远程方法调用。

Binder连接池的主要作用就是将每个业务模块的Binder请求统一转发到远程Service中执行，从而避免重复创建Service的过程。

![](http://i.imgur.com/d5OMZwA.jpg)

核心Code

# 六、选择合适的IPC方式

![](http://i.imgur.com/x7EfgS9.jpg)


> [使用AIDL实现进程间的通信之复杂类型传递](http://blog.csdn.net/liuhe688/article/details/6409708)

> [Java并发编程：并发容器之CopyOnWriteArrayList ](http://ifeve.com/java-copy-on-write/)

> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)