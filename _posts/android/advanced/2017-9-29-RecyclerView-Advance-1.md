---
layout: post
title: RecyclerView封装 一
date: 2017-9-29
excerpt: "RecyclerView封装 一"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

先帖效果图，我们想实现的一系列效果：

- Header,Footer

    ![](https://i.imgur.com/w6rp5sD.png)

    ![](https://i.imgur.com/fATcYUi.png)

- Endless Footer

    ![](https://i.imgur.com/jl9baMT.jpg)

    ![](https://i.imgur.com/5jcGgJt.jpg)
    
    ![](https://i.imgur.com/1JwjYs6.jpg)


# 封装一、 创建RecyclerView，给他点数据

这里我们使用的MVP架构。如果对这个架构不熟悉可以先看看[这里](http://vivianking6855.github.io/2017/09/08/Android-Architecture/)

目录结构如下：

![](https://i.imgur.com/67RloFT.jpg)


下面我一步一步来实现

## 1） 创建 HugeRecyclerView extends RecyclerView

    public class HugeRecyclerView extends RecyclerView {
    
        public HugeRecyclerView(Context context) {
            super(context);
            init(null, 0);
        }
    
        public HugeRecyclerView(Context context, AttributeSet attrs) {
            super(context, attrs);
            init(attrs, 0);
        }
    
        public HugeRecyclerView(Context context, AttributeSet attrs, int defStyle) {
            super(context, attrs, defStyle);
            init(attrs, defStyle);
        }
    
        private void init(AttributeSet attrs, int defStyle) {
            // Load attributes
        }
    }
        
## 2）model

    public class SampleDataApi {
    
        public static List<SampleModel> getData() {
            // do your consume work here
            SystemClock.sleep(2000);
    
            return mock();
        }
    
        private static List<SampleModel> mock() {
            List<SampleModel> list = new ArrayList<>();
            for (int i = 'A'; i <= 'Z'; i++) {
                SampleModel model = new SampleModel("" + (char) i);
                list.add(model);
            }
    
            return list;
        }
    }
    
    public class SampleModel {
        public String mTitle;
    
        SampleModel(String title) {
            mTitle = title;
        }
    }
        

## 3） presenter

    public class SamplePresenter {
        private static final String TAG = "SecondPresenter";
        private ISampleListener mListener;
        // rx
        private CompositeDisposable compositeDisposable = new CompositeDisposable();
    
        public void setListener(ISampleListener listener) {
            mListener = listener;
        }
    
        public void getData() {
            Log.d(TAG, "getData");
            // load start notify
            if (mListener != null) {
                mListener.OnLoadStart();
            }
            // load data
            Disposable dispose = Observable.fromCallable(SampleDataApi::getData)
                    .subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(data -> {
                                Log.d(TAG, "getData Success" + StringUtils.join(data.toArray(), ","));
                                if (mListener != null) {
                                    mListener.OnLoadSuccess(data);
                                }
                            },
                            error -> {
                                Log.w(TAG, "getData error:", error);
                                if (mListener != null) {
                                    mListener.OnLoadFail(error.toString());
                                }
                            });
    
            compositeDisposable.add(dispose);
        }
    
        public void destroy() {
            // to avoid okhttp leak
            if (compositeDisposable != null && !compositeDisposable.isDisposed()) {
                compositeDisposable.dispose();
            }
        }
    
    
    }
    
   里面用到了retrolambda和rxjava，rxandroid，记得配置gradle
   
## 4）Adapter

       public class SampleRecyclerAdapter extends RecyclerView.Adapter {
        private Context mContext;
    
        // model data
        protected List<SampleModel> mItemList;
    
        public SampleRecyclerAdapter(Context context) {
            mContext = context;
            mItemList = new ArrayList<>();
        }
    
        public void setData(List<SampleModel> data) {
            mItemList.clear();
            mItemList.addAll(data);
            notifyDataSetChanged();
        }
    
        public void addData(List<SampleModel> data) {
            int len = mItemList.size();
            mItemList.addAll(data);
            notifyItemChanged(len);
        }
    
        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return new ItemViewHolder(LayoutInflater.from(mContext)
                    .inflate(R.layout.recycler_item, parent, false));
        }
    
        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
            ((ItemViewHolder) holder).hint.setText(mItemList.get(position).mTitle);
        }
    
        @Override
        public int getItemCount() {
            return mItemList == null ? 0 : mItemList.size();
        }
    
        class ItemViewHolder extends RecyclerView.ViewHolder {
            private TextView hint;
    
            ItemViewHolder(View view) {
                super(view);
                hint = (TextView) view.findViewById(R.id.tv_content);
            }
        }
    }
    
这是最基本的用法

## 5）xml

activity_main.xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
    
        <TextView
            android:id="@+id/tv_hint"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="@dimen/text_size_medium" />
    
        <com.open.hugerecyclerview.HugeRecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    
    </LinearLayout>
    
recycler_item.xml

    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    
        <TextView
            android:id="@+id/tv_content"
            android:layout_width="match_parent"
            android:layout_height="@dimen/recycler_item_first_h"
            android:gravity="center" />
    </FrameLayout>


## 6）SimpleRecyclerActivity

    public class SimpleRecyclerActivity extends BaseActivity implements ISampleListener {
        private SamplePresenter mPresenter;
        private TextView mHint;
    
        private SampleRecyclerAdapter mAdapter;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
        }
    
        @Override
        protected void initData() {
            mPresenter = new SamplePresenter();
            mPresenter.setListener(this);
        }
    
        @Override
        protected void initView(Bundle savedInstanceState) {
            setContentView(R.layout.activity_simple);
    
            RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
            LinearLayoutManager lm = new LinearLayoutManager(this);
            mRecyclerView.setLayoutManager(lm);
            mAdapter = new SampleRecyclerAdapter(this);
            mRecyclerView.setAdapter(mAdapter);
    
            mHint = (TextView) findViewById(R.id.tv_hint);
        }
    
        @Override
        protected void loadData() {
            mPresenter.getData();
        }
    
        @Override
        public void OnLoadStart() {
            mHint.setText(getString(R.string.load_start));
        }
    
        @Override
        public void OnLoadSuccess(List<SampleModel> data) {
            mHint.setText(getString(R.string.load_success));
            mAdapter.setData(data);
        }
    
        @Override
        public void OnLoadFail(String error) {
            mHint.setText(getString(R.string.load_fail) + error);
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            mPresenter.destroy();
        }
    }

## 看下显示效果

![](https://i.imgur.com/QaDn8MR.png)


# RecyclerView封装系列

RecycleView系列一步步实现了上面的效果

- [RecyclerView封装 一](http://vivianking6855.github.io/2017/09/29/RecyclerView-Advance-1/)
- [RecyclerView封装 二](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-2/)
- [RecyclerView封装 三](http://vivianking6855.github.io/2017/10/09/RecyclerView-Advance-3/)
- [RecyclerView封装 四](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-4/)
- [RecyclerView封装 五](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-5/)

# [Sample Code下载](https://github.com/vivianking6855/android-library/tree/master/HugeRecyclerView)

