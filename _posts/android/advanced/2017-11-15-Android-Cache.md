---
layout: post
title: 数据缓存
date: 2017-11-15
excerpt: "数据缓存"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

在Project中有大量数据显示需求时，数据缓存就会显得尤为重要。

特别是图片等特别消耗内存的数据。不过有非常多的优秀轮子可以实现图像的加载和缓存，例如

- Picasso Square公司
- Fresco Facebook
- Glide ： 2014年google I/O大会上发布的官方推荐([Glide和Picasso对比](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0327/2650.html))


今天这里主要来讨论LRU在普通数据上的使用，比如用于大批量的数据缓存。

解决方案：Global全局存储 + LruCache + [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) 

# 案例需求说明

随机产生一组庞大的数据，首页启动时加载。

加载完毕后，存入内存缓存和Disk缓存。

点击按钮分别从不同的缓存中读取。

# 数据存储设计方案

缓存有三层

- GlobalManager 全局存储，多用于共享少量数据。比如config 配置，user info，状态等
- LruCacheManager 内存缓存，多用于大量数据存储，比如联络人信息，新闻数据等
- DiskCacheManager Disk缓存，多用于大量数据存储，比如联络人信息，新闻数据等

其中LruCacheManager 和 DiskCacheManager都是Lru思想。

# 方案实现：

Project采用MVP框架，所有的数据读取逻辑放在DownloadPresenter中，数据加载放在model的DataApi中。

1. UserApplication中初始化和释放缓存资源

        public class UserApplication extends Application {

            /**
             * 在创建应用程序时调用，可以在这里实例化应用程序的单例，
             * 创建和实例化任何应用程序状态变量或共享资源等
             */
            @Override
            public void onCreate() {
                super.onCreate();
        
                // init cache
                GlobalManager.INSTANCE.init();
                CacheManager.INSTANCE.init(this);
            }
        
            /**
             * 程序终止时调用，如果需要及时的释放资源可以在各个Activity中。
             * onTerminate只有在程序终止时才会调用，程序退到后台时有可能不会终止。
             */
            @Override
            public void onTerminate() {
                super.onTerminate();
        
                GlobalManager.INSTANCE.release();
                CacheManager.INSTANCE.release();
            }
            ......
        }

        -------------------------------------CacheManager
        
        public enum  CacheManager {
            INSTANCE;
        
            public void init(Context context){
                DiskCacheManager.INSTANCE.init(context);
                DiskLruCacheUtils.setDiskLruCache(DiskCacheManager.INSTANCE.getDiskLruCache(context));
            }
        
            public void release(){
                DiskCacheManager.INSTANCE.release();
            }
        
        }
        
        ---------------------------------------GlobalManager
        
        public enum GlobalManager {
            INSTANCE;
        
            public int mDataCount;
        
            public void init(){
        
            }
        
            public void release(){
        
            }
        }

2. DownloadPresenter使用RxAndroid框架异步加载数据，并提供缓存读取，写入接口

        public class DownloadPresenter {
            private static final String TAG = "DownloadPresenter";
            private IDownloadListener mListener;
            // rx
            private CompositeDisposable compositeDisposable = new CompositeDisposable();
        
            private Context mContext;
        
            public DownloadPresenter(Context context) {
                mContext = context;
            }
        
            public void setListener(IDownloadListener listener) {
                mListener = listener;
            }
        
            public void loadData() {
                Log.d(TAG, "loadData");
                // load start notify
                if (mListener != null) {
                    mListener.OnLoadStart();
                }
                // load data
                Disposable dispose = Observable.fromCallable(DataApi::getData)
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(data -> {
                                    //Log.d(TAG, "loadData Success: " + StringUtils.join(data.toArray(), ","));
                                    if (mListener != null) {
                                        mListener.OnLoadSuccess(data);
                                    }
        
                                    // add to disk cache
                                    cacheToDisk(data);
                                    // add to lru cache
                                    cacheToLru(data);
        
                                    GlobalManager.INSTANCE.mDataCount = data.size();
                                },
                                error -> {
                                    Log.w(TAG, "loadData error: ", error);
                                    if (mListener != null) {
                                        mListener.OnLoadFail(error.toString());
                                    }
                                });
        
                compositeDisposable.add(dispose);
            }
        
            private void cacheToDisk(List<DataModel> list) {
                for (DataModel data : list) {
                    DiskLruCacheUtils.set(data.id, data.description);
                }
            }
        
            private void cacheToLru(List<DataModel> list) {
                for (DataModel data : list) {
                    LruCacheManager.INSTANCE.getLruCache(mContext).put(data.id, data.description);
                }
            }
        
            public String getFromDiskCache(String key) {
                return DiskLruCacheUtils.getString(key);
            }
        
            public String getFromLruCache(String key) {
                return LruCacheManager.INSTANCE.getLruCache(mContext).get(key);
            }
        
            public String getFromCache(String key) {
                String str = getFromLruCache(key);
                if (str != null) {
                    Log.d(TAG,"getFromCache is from lru cache");
                    return str;
                }
                Log.d(TAG,"getFromCache is from disk lru cache");
                return getFromDiskCache(key);
            }
        
            public void destroy() {
                // to avoid okhttp leak
                if (compositeDisposable != null && !compositeDisposable.isDisposed()) {
                    compositeDisposable.dispose();
                }
            }
        
        }

3. MainActivity负责显示缓存读取数据，以及UI交互逻辑

        public class MainActivity extends BaseActivity implements IDownloadListener {
            private final static String TAG = "MainActivity";
        
            // presenter of MVP
            private DownloadPresenter mPresenter;
        
            private Context mContext;
            private TextView mHint;
        
            // permission request code
            private final static int PERMISSION_REQUEST_CODE_RECORD = 10000;
            // if system request dialogue show
            private boolean mSystemPermissionShowing = false;
            private AlertDialog mNeverShowDlg;
        
            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
            }
        
            @Override
            protected void initData() {
                mContext = this;
                mPresenter = new DownloadPresenter(this);
                mPresenter.setListener(this);
            }
        
            @Override
            protected void initView(Bundle savedInstanceState) {
                setContentView(R.layout.activity_main);
        
                mHint = (TextView) findViewById(R.id.tv_hint);
                mHint.setMovementMethod(ScrollingMovementMethod.getInstance());
        
                findViewById(R.id.tv_lru).setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mHint.setText(mPresenter.getFromLruCache("a"));
                    }
                });
                findViewById(R.id.tv_disk).setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mHint.setText(mPresenter.getFromDiskCache("c"));
                    }
                });
                findViewById(R.id.tv_common).setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mHint.setText(GlobalManager.INSTANCE.mDataCount + ", b: " +  mPresenter.getFromCache("b"));
                    }
                });
        
            }
        
            @Override
            protected void loadData() {
        
            }
        
            @Override
            protected void onPause() {
                super.onPause();
                DiskCacheManager.INSTANCE.flushDiskLruCache();
            }
        
            @Override
            public void OnLoadStart() {
        
            }
        
            @Override
            public void OnLoadSuccess(List<DataModel> data) {
                Toast.makeText(mContext, "OnLoadSuccess " + data.size(), Toast.LENGTH_LONG).show();
            }
        
            @Override
            public void OnLoadFail(String error) {
        
            }
        
            @Override
            protected void onResume() {
                super.onResume();
                dealWithPermission();
            }
        
            @Override
            protected void onDestroy() {
                super.onDestroy();
                if (mNeverShowDlg != null && mNeverShowDlg.isShowing()) {
                    mNeverShowDlg.dismiss();
                }
        
                mPresenter.destroy();
            }
        
            @Override
            public void onRequestPermissionsResult(int requestCode, String[] permissions,
                                                   int[] grantResults) {
                if (requestCode == PERMISSION_REQUEST_CODE_RECORD) {
                    mSystemPermissionShowing = false;
                    int hasWriteStoragePermission = ContextCompat.checkSelfPermission(MainActivity.this,
                            Manifest.permission.WRITE_EXTERNAL_STORAGE);
                    if (hasWriteStoragePermission != PackageManager.PERMISSION_GRANTED) {
                        // if user choose deny,exit app
                        finish();
                    }
                }
                super.onRequestPermissionsResult(requestCode, permissions, grantResults);
            }
        
            private void dealWithPermission() {
                int hasWriteStoragePermission = ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
                if (hasWriteStoragePermission != PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "permission not granted");
                    // if user choose never show before, request system permission will not work
                    boolean never_show = !ActivityCompat.shouldShowRequestPermissionRationale(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
                    if (never_show) {
                        Log.d(TAG, "user choose never show");
                        showNeverShowHintDialogue();
                    } else {
                        if (!mSystemPermissionShowing) {
                            Log.d(TAG, "show system permission dialogue");
                            // show system permission dialogue
                            mSystemPermissionShowing = true;
                            showSystemRequestDialog();
                        }
                    }
                } else {
                    mPresenter.loadData();
                }
            }
        
            private void showSystemRequestDialog() {
                // show system permission dialogue
                ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                        PERMISSION_REQUEST_CODE_RECORD);
            }
        
            private void showNeverShowHintDialogue() {
                mNeverShowDlg = new AlertDialog.Builder(mContext)
                        .setCancelable(false)
                        .setTitle(getString(R.string.grant_permission_title))
                        .setMessage(getString(R.string.grant_permission))
                        .setPositiveButton(getString(R.string.grant_setting), new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                // user choose never show,you could change your resolution here for your project
                                gotoAppDetail();
                            }
                        })
                        .setNegativeButton(getString(R.string.grant_cancel), new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                finish();
                            }
                        })
                        .show();
            }
        
            private void gotoAppDetail() {
                Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                intent.addCategory(Intent.CATEGORY_DEFAULT);
                intent.setData(Uri.parse("package:" + getPackageName()));
                startActivity(intent);
            }
        
        }


[Code Github下载](https://github.com/vivianking6855/android-advanced/tree/master/CacheDemo)


# Reference

- [Android DiskLruCache完全解析](http://blog.csdn.net/guolin_blog/article/details/28863651)
- [Android之本地缓存——LruCache（内存缓存）与DiskLruCache（硬盘缓存）统一框架](http://blog.csdn.net/yangzhaomuma/article/details/51810723)
- [三分钟学会缓存工具DiskLruCache](http://blog.csdn.net/u012702547/article/details/47276385)
