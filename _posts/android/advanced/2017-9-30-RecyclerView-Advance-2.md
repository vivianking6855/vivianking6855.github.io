---
layout: post
title: RecyclerView封装 二
date: 2017-9-30
excerpt: "RecyclerView封装 二"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}


# 2. 加上Header和Footer

我们希望RecyclerView可以支持加入多个Header和Footer，虽然比较常用的是单个Header和Footer

这个需求需要从Adapter入手，增加addHeader or addFooter入口。下面我们来改动BaseRecyclerAdapter

先来定义一些变量，主要是Header List, size 和map方便我们操作他们。

    //header view and footer view can't be recycled
    private List<RecyclerView.ViewHolder> mHeaderViews = new ArrayList<>();
    //store the ViewHolder for remove header view
    private Map<View, RecyclerView.ViewHolder> mHeaderViewHolderMap = new HashMap<>();
    private int mHeaderSize;
    //header view and footer view can't be recycled
    private List<RecyclerView.ViewHolder> mFooterViews = new ArrayList<>();
    //store the ViewHolder for remove footer view
    private Map<View, RecyclerView.ViewHolder> mFooterViewHolderMap = new HashMap<>();
    private int mFooterSize;


想要支持多个type，关键是处理好getItemViewType和onCreateViewHolder.

这两个分别是标注每个item的类别，以及依据不同的类别来创建对应的ViewHolder


    @Override
    public int getItemViewType(int position) {
        // no header and footer or normal type
        if (isContentView(position)) {
            return TYPE_NORMAL;
        }
        return position;
    }
    
        @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        // normal
        if (TYPE_NORMAL == viewType) {
            return onCreateItemViewHolder(parent, viewType);
        }
        if (viewType < mHeaderSize) {// header
            return mHeaderViews.get(viewType);
        }
        // footer
        return mFooterViews.get(viewType - getBasicItemCount() - mHeaderSize);
    }
    
    private boolean isContentView(int position) {
        return position >= mHeaderSize && position < (getBasicItemCount() + mHeaderSize);
    }

    private int getBasicItemCount() {
        return mItemList == null ? 0 : mItemList.size();
    }
    
看看我们的修改

- 在getItemViewType，我们判断如果是正常的内容就返回TYPE_NORMAL，header or footer我们就返回 position作为type
- 在onCreateViewHolder，如果是TYPE_NORMAL我们就返回正常的ViewHolder，如果是header or footer我们就依据position返回对应的header or footer

另外，我们要给外部提供接口可以操作header和footer

    public void addHeaderView(View view) {
            RecyclerView.ViewHolder holder = new RecyclerView.ViewHolder(view) {
            };
            holder.setIsRecyclable(false);
            mHeaderViewHolderMap.put(view, holder);
            mHeaderViews.add(holder);
            mHeaderSize = mHeaderViews.size();
            notifyItemInserted(mHeaderSize - 1);
        }
    
        public void removeHeaderView(View view) {
            RecyclerView.ViewHolder viewHolder = mHeaderViewHolderMap.get(view);
            if (mHeaderViews.remove(viewHolder)) {
                mHeaderSize = mHeaderViews.size();
                mHeaderViewHolderMap.remove(view);
                notifyDataSetChanged();
            }
        }
    
        public void addFooterView(View view) {
            RecyclerView.ViewHolder holder = new RecyclerView.ViewHolder(view) {
            };
            holder.setIsRecyclable(false);
            mFooterViewHolderMap.put(view, holder);
            mFooterViews.add(holder);
            mFooterSize = mFooterViews.size();
            notifyItemInserted(mFooterSize - 1);
        }
    
        public void removeFooterView(View view) {
            RecyclerView.ViewHolder viewHolder = mFooterViewHolderMap.get(view);
            if (viewHolder != null && mFooterViews.remove(viewHolder)) {
                mFooterSize = mFooterViews.size();
                mFooterViewHolderMap.remove(view);
                notifyDataSetChanged();
            }
        }
    }
    
准备Okay，我们来用吧

首先创建两个xml存放header和footer的布局，我这放了两个一样的，字串改了下

    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/holo_blue_light">
    
        <TextView
            android:id="@+id/header"
            android:layout_width="match_parent"
            android:layout_height="@dimen/recycler_item_h"
            android:gravity="center"
            android:text="Header" />
    </FrameLayout>
    
    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/holo_blue_light">
    
        <TextView
            android:id="@+id/footer"
            android:layout_width="match_parent"
            android:layout_height="@dimen/recycler_item_h"
            android:gravity="center"
            android:text="Footer" />
    </FrameLayout>

HeaderFooterActivity 同 SimpleRecyclerActivity基本一样，我们加上两句

    @Override
    protected void initView(Bundle savedInstanceState) {
        setContentView(R.layout.activity_head_footer);

        RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
        LinearLayoutManager lm = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(lm);
        mAdapter = new SampleRecyclerAdapter(this);
        mRecyclerView.setAdapter(mAdapter);

        // add header and foot for RecyclerView ******************
        mAdapter.addHeaderView(getLayoutInflater().inflate(R.layout.recycler_header, mRecyclerView, false));
        mAdapter.addHeaderView(getLayoutInflater().inflate(R.layout.recycler_header, mRecyclerView, false));
        mAdapter.addHeaderView(getLayoutInflater().inflate(R.layout.recycler_header, mRecyclerView, false));
        mAdapter.addFooterView(getLayoutInflater().inflate(R.layout.recycler_footer, mRecyclerView, false));
        mAdapter.addFooterView(getLayoutInflater().inflate(R.layout.recycler_footer, mRecyclerView, false));

        mHint = (TextView) findViewById(R.id.tv_hint);
    }
    
看星星下面的几句，使用就是这么简单。

来看看效果：

![](https://i.imgur.com/w6rp5sD.png)

![](https://i.imgur.com/fATcYUi.png)

# RecyclerView封装系列

- [RecyclerView封装 一](http://vivianking6855.github.io/2017/09/29/RecyclerView-Advance-1/)
- [RecyclerView封装 二](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-2/)
- [RecyclerView封装 三](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-3/)
- [RecyclerView封装 四](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-4/)
- [RecyclerView封装 五](http://vivianking6855.github.io/2017/09/30/RecyclerView-Advance-5/)


# [Sample Code下载](https://github.com/vivianking6855/android-library/tree/master/HugeRecyclerView)


