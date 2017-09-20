---
layout: post
title: Android框架 之 MVC, MVP, MVVM
date: 2017-9-8
excerpt: "Android框架 之 MVC, MVP, MVVM"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

介绍MVC, MVP, MVVM三者的区别。

同时用实例说明。

实例功能说明：从网站下载cofig数据，显示在页面上

# 1. MVC (Model View Controller)

MVC（Model View Controller）是软件框架中最常见的一种框架。

![](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvc.png)

## 工作原理

当用户触发事件后，view层会发送指令到controller层。

接着controller去通知model层更新数据，model层更新完数据以后直接显示在view层上。

在Android上可以理解为：

- Controller： Activity
- View: layout.xml
- Model: 存储数据和一些具体的逻辑操作。（实例中的DownloadApi, UrlConfig)

## 实例解析

activity_main.xml，两个控件，一个用来点击下载，一个用来显示数据

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_margin="20dp"
        android:gravity="center_horizontal"
        android:orientation="vertical">
    
        <TextView
            android:id="@+id/tv_download"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Download" />
    
        <TextView
            android:id="@+id/tv_result"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    
    </LinearLayout>

MainActivity.java

    public class MainActivity extends BaseActivity {
        private static final String TAG = "MainActivity";
    
        private ProgressDialog mProcessDlg;
        private TextView mTVResult;
    
        // website
        private CompositeDisposable compositeDisposable = new CompositeDisposable();
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
        }
    
        @Override
        protected void initData() {
    
        }
    
        @Override
        protected void initView(Bundle savedInstanceState) {
            setContentView(R.layout.activity_main);
    
            mTVResult = (TextView) findViewById(R.id.tv_result);
    
            findViewById(R.id.tv_download).setOnClickListener(v -> {
                showProgress();
                DownloadApi engine = (WebsiteApi) DownloadApi.INSTANCE;
                Disposable config = Observable.fromCallable(() -> engine.getConfig())
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(urlConfig -> {
                                    final String result = engine.getGson().toJson(urlConfig.Debug).toString();
                                    Log.d(TAG, "" + result);
                                    mTVResult.setText(result);
                                },
                                error -> {
                                    dismissProgress();
                                },
                                () -> {
                                    Log.d(TAG, "tv_download complete");
                                    dismissProgress();
                                });
                compositeDisposable.add(config);
            });
    
        }
  
        ......
    
        public void showProgress() {
            if (mProcessDlg == null) {
                mProcessDlg = ProgressDialog.show(this, "download", "I'm loading, please wait...");
            }
        }
    
        public void dismissProgress() {
            if (mProcessDlg != null) {
                mProcessDlg.dismiss();
            }
        }

    }

DownloadApi.java

    public enum DownloadApi {
        INSTANCE;
        private static final String TAG = "DownloadApi";
    
        private final OkHttpClient mOkHttpClient = new OkHttpClient();
        public final Gson mGson = new Gson();
    
        public static Gson getGson() {
            return INSTANCE.mGson;
        }
    
        /**
         * get config sync through okhttp
         */
        public UrlConfig getConfig() {
            try {
                Request request = new Request.Builder()
                        .url(Const.URL_CONFIG)
                        .build();
    
                Response response = mOkHttpClient.newCall(request).execute();
                if (!response.isSuccessful()) {
                    Log.w(TAG, "getConfig failed " + response);
                    return null;
                }
                Log.w(TAG, "getConfig " + response.body());
                UrlConfig config = mGson.fromJson(response.body().charStream(), UrlConfig.class);
                return config;
            } catch (Exception ex) {
                Log.w(TAG, "getConfig ex:", ex);
            }
    
            return null;
        }
    
    }

 
我们从MVC角度分析下Code：

- xml中有布局代码 （View）
- activity作为一个Controller，里面的逻辑是监听用户点击并作出相应的操作，比如调用是调用WebsiteApi的方法去获取数据。
- WebsiteApi，UrlConfig等类,则表示MVC中的model层，里面是数据和一些具体的逻辑操作

## 缺陷

从实例Code可以看出一些MVC的缺陷

- activity的责任过重，既是controller又是view
    - 比如说有一个ProgressDialog的更新，其实是view层的逻辑，但是没办法写到xml里面
    - 比如TextView.setTextView()
    - 若是逻辑很复杂的页面，activity或者fragment是动辄上千行，维护起来相当累
- 另外还有，view层和model层是相互可知的，两层之间存在耦合。对于一个大型程序来说是非常致命的，因为这表示开发，测试，维护都需要花大量的精力。

# 2. MVP(Model View Presenter)

MVP作为MVC的演化，解决了MVC不少的缺点。

![](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvp.png)

## 工作原理

view层发出的事件传递到presenter层中，presenter层去操作model层。

并且将数据返回给view层，整个过程中view层和model层完全没有联系。

在Android上可以理解为：

- MVP的model层相对于MVC是一样的
- activity和fragment不再是controller层，而是纯粹的view层。
    - activity，fragment可以实现定义好的接口，在对应的presenter中通过接口调用方法。
- 所有关于用户事件的转发全部交由presenter层处理。

相较MVC最明显的差别就是view层和model层完全的解耦，取而代之的presenter层充当了桥梁的作用。

虽然view层和model层解耦了，但是view层和presenter层看起来耦合在一起了。其实不是的，对于view层和presenter层的通信，我们是可以通过接口实现。

最好的方式是：

- fragment作为view层
- activity则是用于创建view层(fragment)和presenter层(presenter)的一个控制器

还可以编写测试用的View，模拟用户的各种操作，从而实现对Presenter的测试。这就解决了MVC模式中测试，维护难的问题。

## 实例解析

activity_main.xml，两个控件，一个用来点击下载，一个用来显示数据。跟MVC一样

MainActivity

     public class MainActivity extends BaseActivity implements IDownloadListener {
        private static final String TAG = "MainActivity";
    
        private ProgressDialog mProcessDlg;
        private TextView mTVResult;
        private DownloadPresenter mPresenter;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
        }
    
        @Override
        protected void initData() {
            mPresenter = new DownloadPresenter();
            mPresenter.setIDownloadListener(this);
        }
    
        @Override
        protected void initView(Bundle savedInstanceState) {
            setContentView(R.layout.activity_main);
    
            mTVResult = (TextView) findViewById(R.id.tv_result);
            findViewById(R.id.tv_download).setOnClickListener(v -> {
                showProgress();
                mPresenter.start();
            });
        }
    
        @Override
        protected void loadData() {
    
        }
    
        public void showProgress() {
            if (mProcessDlg == null) {
                mProcessDlg = ProgressDialog.show(this, "download", "I'm loading, please wait...");
            }
        }
    
        public void dismissProgress() {
            if (mProcessDlg != null) {
                mProcessDlg.dismiss();
            }
        }
    
        @Override
        public void onLoadStart() {
            showProgress();
        }
    
        @Override
        public void onLoadSuccess(String result) {
            dismissProgress();
            mTVResult.setText(result);
        }
    
        @Override
        public void onLoadFail(String error) {
            dismissProgress();
            mTVResult.setText("load error: " + error);
        }
    }

DownLoadApi.java跟MVC中的一样

DownloadPresenter.java

    public class DownloadPresenter {
        private static final String TAG = "DownloadPresenter";
    
        private IDownloadListener mLoadListener;
    
        // website
        private CompositeDisposable compositeDisposable = new CompositeDisposable();
    
        public void setIDownloadListener(IDownloadListener listener) {
            mLoadListener = listener;
        }
    
        public void get() {
    
        }
    
        public void start() {
            if (mLoadListener != null) {
                mLoadListener.onLoadStart();
            }
            DownLoadApi engine = (DownLoadApi) DownLoadApi.INSTANCE;
            Disposable config = Observable.fromCallable(() -> engine.getConfig())
                    .subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(urlConfig -> {
                                final String result = engine.getGson().toJson(urlConfig).toString();
                                Log.d(TAG, "" + result);
                                if (mLoadListener != null) {
                                    mLoadListener.onLoadSuccess(result);
                                }
                            },
                            error -> {
                                Log.w(TAG, "tv_download error:", error);
                                if (mLoadListener != null) {
                                    mLoadListener.onLoadFail(error.toString());
                                }
                            },
                            () -> {
                                Log.d(TAG, "tv_download complete");
                            });
            compositeDisposable.add(config);
        }
    
        public void destroy() {
            // to avoid okhttp leak
            if (compositeDisposable != null && !compositeDisposable.isDisposed()) {
                compositeDisposable.dispose();
            }
        }
    
    }

我们从MVP角度分析下Code：

- 把activity的跟model层相关的逻辑抽取出来放在Presenter里面
- Presenter里面在相应的时机调用接口，activity实现接口更新View

和MVC最大的不同是：MVP把activity作为了view层，整个activity没有任何和model层相关的逻辑代码。

取而代之的是把代码放到了presenter层中，presenter获取了model层的数据之后，通过接口的形式将view层需要的数据返回给它。

这样做到的好处：

- activity的代码逻辑减少了
- view层和model层完全解耦，也更方便测试
    - 比如你需要测试一个http请求是否顺利，你不需要写一个activity，只需要写一个java类，实现对应的接口，presenter获取了数据自然会调用相应的方法
    - 比如你也可以自己在presenter中mock数据，分发给view层，用来测试布局是否正确。

有个开源的库可以轻松实现MVP，有兴趣可以研究看看[Mosby](https://github.com/sockeqwe/mosby)： Model-View-Presenter and Model-View-Intent library for Android apps.

# 3. MVVM (Model-View-ViewModel)

Model-View-ViewModel 就是将其中的 View 的状态和行为抽象化，让我们可以将UI和业务逻辑分开。

这些 ViewModel 已经帮我们做了，它可以取出 Model 的数据同时帮忙处理 View 中由于需要展示内容而涉及的业务逻辑。

![](https://camo.githubusercontent.com/bf85a9dad3d3c42767797ccd35eb1bfcab654c8a/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f313630302f312a564c68585552484c3972476c784e59653979647156672e706e67)

google 推出了[databinding](https://bintray.com/android/android-tools/com.android.databinding.dataBinder) ，使得MVVM的使用更方便了

## 工作原理

MVVM模式是通过以下三个核心组件组成：

- Model - 包含了业务和验证逻辑的数据模型
- View - 定义屏幕中View的结构，布局和外观
- ViewModel - 扮演“View”和“Model”之间的使者，帮忙处理 View 的全部业务逻辑

view层和viewmodel层是相互绑定的，更新viewmodel层的数据，view层会相应的变动ui。

## 实例解析

gradle配置(2.3.3) 

app build.gradle 中加入：

    android {
    ......
    
        dataBinding {
            enabled true
        }
    
    }

activity_main.xml

    <?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
    
        <data>
    
            <variable
                name="result"
                type="com.vv.mvp.model.DownloadResult" />
        </data>
    
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_margin="20dp"
            android:gravity="center_horizontal"
            android:orientation="vertical">
    
            <TextView
                android:id="@+id/tv_download"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Download" />
    
            <TextView
                android:id="@+id/tv_result"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="50dp"
                android:text="@{result.result}" />
    
        </LinearLayout>
    </layout>


DownloadResult.java (data binding class)

    public class DownloadResult {
        public ObservableField<String> result = new ObservableField<>();
    }

MainActivity.java (MVP的presenter没有移除）

    public class MainActivity extends BaseActivity implements IDownloadListener {
        private static final String TAG = "MainActivity";
    
        private ProgressDialog mProcessDlg;
        private DownloadPresenter mPresenter;
    
        private ActivityMainBinding mBinding;
        private DownloadResult mResult;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
        }
    
        @Override
        protected void initData() {
            mPresenter = new DownloadPresenter();
            mPresenter.setIDownloadListener(this);
        }
    
        @Override
        protected void initView(Bundle savedInstanceState) {
            mBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
            findViewById(R.id.tv_download).setOnClickListener(v -> {
                showProgress();
                mPresenter.start();
            });
        }
    
        @Override
        protected void loadData() {
    
        }
    
        public void showProgress() {
            if (mProcessDlg == null) {
                mProcessDlg = ProgressDialog.show(this, "download", "I'm loading, please wait...");
            }
        }
    
        public void dismissProgress() {
            if (mProcessDlg != null) {
                mProcessDlg.dismiss();
            }
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
    
            // to avoid dlg leak
            dismissProgress();
    
            mPresenter.destroy();
        }
    
        @Override
        public void onLoadStart() {
            showProgress();
        }
    
        @Override
        public void onLoadSuccess(String result) {
            dismissProgress();
            mResult.result.set(result);
        }
    
        @Override
        public void onLoadFail(String error) {
            dismissProgress();
            mResult.result.set("load error: " + error);
        }
    }


我们从MVVM角度分析下Code：

- 把view的更新交给binding来处理（ViewModel）

这里只是最简单的binding，实际上搭配recyclerview等可以省去非常多的view状态更新的code.


# 小结

- 若项目不是非常复杂，使用android自己MVC问题不大。
- 若是项目逻辑和模块较为复杂，为了避免Activity过于庞大，MVP是一个不错的选择。
- MVP和MVVM一起使用对复杂和界面状态和行为逻辑复杂的项目，可以很大的降低耦合和维护成本。

[Sample Code下载](https://github.com/vivianking6855/android-template/tree/master/android-architecture)

# Reference

以上图片均是从下面参看地址摘取，若有任何问题，请通知下，我会立即删除。

> [认清MVC，MVP和MVVM/](http://zjutkz.net/2016/04/13/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC%EF%BC%8CMVP%E5%92%8CMVVM/)

> [Android官方框架DataBinding](http://blog.csdn.net/dabaoonline/article/details/51970916)

> [Android官方数据绑定框架DataBinding(一)](http://blog.csdn.net/qibin0506/article/details/47393725)

> [MVVM 模式介绍](https://github.com/xitu/gold-miner/blob/master/TODO%2Fapproaching-android-with-mvvm.md)
