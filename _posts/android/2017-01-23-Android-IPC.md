---
layout: post
title: 2017-1-23 IPC - Inter-Process Communication 进程间通信
date: 2017-1-23
excerpt: "IPC 进程间通信"
tags: [Android]
comments: true
---

## 一、IPC含义

IPC Inter-Process Communication.含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。


## 二、Android中的多进程模式

在Android中使用多进程只有一种方法：给四大组件（Activity，Service，Receiver,ContentProvider)在AndroidMenifest中制定android:process属性

使用多进程会有下面几个方面的问题：

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharePreference的可靠性下降（SharePreference不支持两个进程同时去执行写操作）
- Application会多次创建

## 三、IPC的基础概念

- 当通过Intent和和Binder传输数据时，需要用到Serializable和Parcelable把数据序列化
- Binder
- Binder实现了IBinder接口
    - 从IPC角度来说Binder是跨进程通信的一种方式
    - 从framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManger等）和形影ManagerService的桥梁
    - 从Android应用层来说，Binder是客户端和服务器端进行通信的媒介
    - Binder主要用在Service中，包括AIDL和Messenger（底层其实是AIDL)

## 四、Android中的IPC方式

跨进程通信方式有很多：Intent传递Bundle数据，共享文件，Binder，Intent传递Bundle数据,ContentProvider,Socket.

下面一一来看下每个方式。

### 1. Intent附加extras传递信息

三大组件（Activity，Service，Receiver)都支持Intent传递Bundle数据（Bundle有实现Parcelable接口）

### 2. 通过共享文件的方式共享数据

- file文件
    - android基于Linux对文件并发读写没有限制
    - 适用于对数据同步要求不高的进程之间的通信
    - 需要处理文件的并发读写问题，特别是并发写（尽量避免或多线程同步来限制多个进程写操作）
- SharedPreferences(不建议用在跨进程间通讯）
    - 目录位于data/data/package name/shared_prefs目录下
    - 本质上也是文件
    - 系统对它的读写有一定的缓存策略，即内存中会有一份缓存，多进模式下读写不可靠

### 3. Messenger - 轻量级的IPC方案

Messenger：信使，可以在不同进程中传递Message对象。是一种轻量级的IPC方案。

- Messenger的底层其实就是AIDL，它对AIDL做了封装，可以更简洁的进行进程间通信
- 一次处理一个请求，虽然不会有线程同步的问题，但是不适合于有大量并发请求的场景

实现Messenger的步骤：

1）服务器端进程

- 创建Service来处理客户端的连接请求
- 创建Handler并通过它创建一个Messenger对象
- Service的onBind中返回Messenger对底层的Binder

2）客户端进程

- bindService
- bind成功后拿到IBinder对象，创建一个Messenger
- 通过Messenger就可以像服务器发送消息，消息类型为Message对象。
- 如果服务器端要回应客户端，就和服务器端一样，传给服务器端一个Messenger
    - 创一个Handler并创建一个新的Messenger
    - 把Messenger通过Message的replayTo参数传给服务端。
    - 服务端通过replyTo参数（Messenger）回应客户端。

### 4. AIDL - 最常用的进程间通讯方式

Messenger是串行的方式处理消息，如果大量的消息同时发送的情景就不太合适了。但是我们可以使用AIDL来实现跨进程的方法调用。

AIDL是最常用的进程间通讯方式，是日常开发中涉及进程间通信是的首选。

AIDL进行进程间通讯的流程

1）服务器端进程

- 创建一个Service用来监听客户端的连接请求
- 创建AIDL文件，将透给客户端的接口在AIDL中声明
- Service中实现这个AIDL接口

2）客户端

- 绑定Service
- bind成功后将Binder对象转成AIDL接口类型
- 调用AIDL接口的方法

3）AIDL接口创建 和 AIDL接口的实现（远程服务器端Service的实现）

[使用AIDL实现进程间的通信之复杂类型传递](http://blog.csdn.net/liuhe688/article/details/6409708)


核心Code


### 5. ContentProvider 

ContentProvider天生支持跨进程数据传递，是Android中提供的专门用于不同应用间进行数据共享的方式。

和Messenger一样，底层实现同样也是Binder.但是使用过程比AIDL简单许多，系统有做封装。

[Android之ContentProvider详解](http://blog.csdn.net/x605940745/article/details/16118939)

### 6. Socket

Socket网络通讯也可以实现数据传递

## 五、Binder连接池

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


## 六、选择合适的IPC方式

![](http://i.imgur.com/x7EfgS9.jpg)


> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)