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


# 封装二、 加上Header和Footer

我们希望RecyclerView可以支持加入多个Header和Footer，虽然比较常用的是单个Header和Footer

这个需求需要从Adapter入手，增加addHeader or addFooter入口。下面我们来改动BaseRecyclerAdapter

## 1) 先来定义一些变量，主要是Header List, size 和map方便我们操作他们。

    //header view and footer view
    private List<View> mHeaderViews = new ArrayList<>();
    private int mHeaderSize;
    private List<View> mFooterViews = new ArrayList<>();
    private int mFooterSize;;


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
            return new ViewHolder(mHeaderViews.get(viewType));
        }
        // footer
        return new ViewHolder(mFooterViews.get(viewType - getBasicItemCount() - mHeaderSize));
    }
    
    public int getBasicItemCount() {
        return mItemList == null ? 0 : mItemList.size();
    }
    
看看我们的修改

- 在getItemViewType，我们判断如果是正常的内容就返回TYPE_NORMAL，header or footer我们就返回 position作为type
- 在onCreateViewHolder，如果是TYPE_NORMAL我们就返回正常的ViewHolder，如果是header or footer我们就依据position返回对应的header or footer

## 2) 外部提供接口可以操作header和footer

    public void addHeaderView(View view) {
        mHeaderViews.add(view);
        mHeaderSize = mHeaderViews.size();
        notifyItemInserted(mHeaderSize - 1);
    }

    public void removeHeaderView(View view) {
        if (mHeaderViews.remove(view)) {
            mHeaderSize = mHeaderViews.size();
            notifyDataSetChanged();
        }
    }

    public List<View> getHeaderView() {
        return mHeaderViews;
    }

    public void addFooterView(View view) {
        mFooterViews.add(view);
        mFooterSize = mFooterViews.size();
        notifyItemInserted(mFooterSize - 1);
    }

    public void removeFooterView(View view) {
        if (mFooterViews.remove(view)) {
            mFooterSize = mFooterViews.size();
            notifyDataSetChanged();
        }
    }

    public List<View> getFooterView() {
        return mFooterViews;
    }
    
[BaseRecyclerAdapter全部Code请看这里](https://github.com/vivianking6855/android-library/tree/master/HugeRecyclerView/hugerecyclerview/src/main/java/com/open/hugerecyclerview/adapter)
      
准备Okay，我们来用吧

## 3）使用

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

HeaderFooterActivity 同 SimpleRecyclerActivity基本一样，我们加上几句,看星星下面的几句，使用就是这么简单。

       @Override
        protected void initView(Bundle savedInstanceState) {
            setContentView(R.layout.activity_head_footer);
    
            RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
            LinearLayoutManager lm = new LinearLayoutManager(this);
            mRecyclerView.setLayoutManager(lm);
            mAdapter = new HeaderFooterRecyclerAdapter(this);
            mRecyclerView.setAdapter(mAdapter);
    
            // add header and foot for RecyclerView ****
            mAdapter.addHeaderView(getLayoutInflater().inflate(R.layout.recycler_header, mRecyclerView, false));
            mAdapter.addHeaderView(getLayoutInflater().inflate(R.layout.recycler_header, mRecyclerView, false));
            mAdapter.addFooterView(getLayoutInflater().inflate(R.layout.recycler_footer, mRecyclerView, false));
            mAdapter.addFooterView(getLayoutInflater().inflate(R.layout.recycler_footer, mRecyclerView, false));
    
            mHint = (TextView) findViewById(R.id.tv_hint);
        }


HeaderFooterRecyclerAdapter继承BaseRecyclerAdapter，再加两个接口setData和addData

       public class HeaderFooterRecyclerAdapter extends
            BaseRecyclerAdapter<SampleModel, HeaderFooterRecyclerAdapter.ItemViewHolder> {
    
            private Context mContext;
        
            public HeaderFooterRecyclerAdapter(Context context) {
                mContext = context;
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
            public ItemViewHolder onCreateItemViewHolder(ViewGroup parent, int viewType) {
                return new ItemViewHolder(LayoutInflater.from(mContext)
                        .inflate(R.layout.recycler_item, parent, false));
            }
        
            @Override
            public void onBindItemViewHolder(ItemViewHolder holder, int position) {
                holder.hint.setText(mItemList.get(position).mTitle);
            }
        
            class ItemViewHolder extends RecyclerView.ViewHolder {
                private TextView hint;
        
                ItemViewHolder(View view) {
                    super(view);
                    hint = (TextView) view.findViewById(R.id.tv_content);
                }
            }
        
        }


## 来看看效果

![](https://i.imgur.com/w6rp5sD.png)

![](https://i.imgur.com/fATcYUi.png)

# [RecyclerView封装目录](http://vivianking6855.github.io/2018/02/24/RecyclerView-Advance-index/)


