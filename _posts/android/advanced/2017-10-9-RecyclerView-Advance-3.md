---
layout: post
title: RecyclerView封装 三
date: 2017-10-9
excerpt: "RecyclerView封装 三"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 封装三、 上滑刷新

思路是找到RecycleView滚动到底的消息，然后给它加上一个footer，依据数据加载的状态来显示不同的footer

核心类说明

- EndlessFooterUtils: 依据数据加载的状态改变底层footer的UI显示
- HugeRecyclerOnScrollListener: 滚动监听，可以在这里处理滚动结束，加载数据的动作
- EndlessFooterView：底层footer view，里面有不同状态对应的UI布局

## 1）先定义个footer view

依据loading, end(no more data), error显示不同的layout

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/loading_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:orientation="vertical">
    
        <ViewStub
            android:id="@+id/loading_viewstub"
            android:layout_width="match_parent"
            android:layout_height="@dimen/endless_footer_h"
            android:layout="@layout/endless_footer_loading" />
    
        <ViewStub
            android:id="@+id/end_viewstub"
            android:layout_width="match_parent"
            android:layout_height="@dimen/endless_footer_h"
            android:layout="@layout/endless_footer_end" />
    
        <ViewStub
            android:id="@+id/error_viewstub"
            android:layout_width="match_parent"
            android:layout_height="@dimen/endless_footer_h"
            android:layout="@layout/endless_footer_error" />
    
    </LinearLayout>

       public class EndlessFooterView extends BaseEndlessFooterView {
    
            public EndlessFooterView(Context context) {
                this(context, null, 0);
            }
        
            public EndlessFooterView(Context context, AttributeSet attrs) {
                this(context, attrs, 0);
            }
        
            public EndlessFooterView(Context context, AttributeSet attrs, int defStyle) {
                super(context, attrs, defStyle);
            }
        
            @Override
            protected void setContentView(Context context) {
                inflate(context, R.layout.endless_footer, this);
            }
        
            @Override
            protected void setLoadingView() {
                if (mLoadingView == null) {
                    ViewStub viewStub = (ViewStub) findViewById(R.id.loading_viewstub);
                    mLoadingView = viewStub.inflate();
                }
            }
        
            @Override
            protected void setErrorView() {
                if (mErrorView == null) {
                    ViewStub viewStub = (ViewStub) findViewById(R.id.error_viewstub);
                    mErrorView = viewStub.inflate();
                }
            }
        
            @Override
            protected void setEndView() {
                if (mTheEndView == null) {
                    ViewStub viewStub = (ViewStub) findViewById(R.id.end_viewstub);
                    mTheEndView = viewStub.inflate();
                }
            }
        
        }

BaseEndlessFooterView主要处理一些view的显示逻辑

    public abstract class BaseEndlessFooterView extends RelativeLayout {
        private final static String TAG = "BaseEndlessFooterView";
    
        // failed view
        protected View mErrorView;
        // end, no more data view
        protected View mTheEndView;
        // loading views
        protected View mLoadingView;
    
        // view state
        protected State mState = State.Normal;
    
        public enum State {
            Normal,    // normal status, not loading, normal show list
            End,       // no more data
            Loading,  // data loading
            Error      // load fail
        }
    
        // set view layout
        protected abstract void setContentView(Context context);
    
        // set loading view
        protected abstract void setLoadingView();
    
        // set error view
        protected abstract void setErrorView();
    
        // set end/no more data view
        protected abstract void setEndView();
    
        public BaseEndlessFooterView(Context context) {
            this(context,null,0);
        }
    
        public BaseEndlessFooterView(Context context, AttributeSet attrs) {
            this(context, attrs,0);
        }
    
        public BaseEndlessFooterView(Context context, AttributeSet attrs, int defStyle) {
            super(context, attrs, defStyle);
            initView(context);
        }
    
        private void initView(Context context) {
            setContentView(context);
            setOnClickListener(null);
            setState(State.Normal);
    
            // inflate views
            setLoadingView();
            setEndView();
            setErrorView();
        }
    
        public State getState() {
            return mState;
        }
    
        /**
         * set endless footer status
         *
         * @param state status of endless footer view
         */
        public void setState(State state) {
            // check status
            if (mState == state) {
                Log.d(TAG, "setState, state not change, return");
                return;
            }
            mState = state;
    
            // hide all first
            hideAllView();
    
            // deal with different state
            switch (state) {
                case Normal:
                    setOnClickListener(null);
                    break;
                case Loading:
                    dealLoading();
                    break;
                case End:
                    dealEnd();
                    break;
                case Error:
                    dealError();
                    break;
                default:
                    break;
            }
        }
    
        private void dealLoading() {
            setOnClickListener(null);
            if (mLoadingView != null) {
                mLoadingView.setVisibility(VISIBLE);
            }
        }
    
        private void dealEnd() {
            setOnClickListener(null);
            if (mTheEndView != null) {
                mTheEndView.setVisibility(VISIBLE);
            }
        }
    
        private void dealError() {
            if (mErrorView != null) {
                mErrorView.setVisibility(VISIBLE);
            }
        }
    
        private void hideAllView() {
            if (mLoadingView != null) {
                mLoadingView.setVisibility(GONE);
            }
            if (mTheEndView != null) {
                mTheEndView.setVisibility(GONE);
            }
            if (mErrorView != null) {
                mErrorView.setVisibility(GONE);
            }
        }
    }

## 2）封装了一个util来控制footer view的status

        public class EndlessFooterUtils {
            private static final String TAG = "EndlessFooterUtils";
            private BaseEndlessFooterView mView;
        
            public EndlessFooterUtils(BaseEndlessFooterView view) {
                mView = view;
            }
        
            /**
             * Sets end.you must set footer view before you use
             *
             * @param recyclerView the recycler view
             * @param pageSize     the page size
             */
            public void setEnd(RecyclerView recyclerView, int pageSize) {
                setFooterViewState(recyclerView, pageSize, BaseEndlessFooterView.State.End, null);
            }
        
            /**
             * Sets normal.you must set footer view before you use
             *
             * @param recyclerView the recycler view
             * @param pageSize     the page size
             */
            public void setNormal(RecyclerView recyclerView, int pageSize) {
                setFooterViewState(recyclerView, pageSize, BaseEndlessFooterView.State.Normal, null);
            }
        
            /**
             * Sets loading.you must set footer view before you use
             *
             * @param recyclerView the recycler view
             * @param pageSize     the page size
             */
            public void setLoading(RecyclerView recyclerView, int pageSize) {
                setFooterViewState(recyclerView, pageSize, BaseEndlessFooterView.State.Loading, null);
            }
        
            /**
             * Sets error.you must set footer view before you use
             *
             * @param recyclerView  the recycler view
             * @param pageSize      the page size
             * @param errorListener error status listener
             */
            public void setError(RecyclerView recyclerView, int pageSize,
                                 View.OnClickListener errorListener) {
                setFooterViewState(recyclerView, pageSize, BaseEndlessFooterView.State.Error, errorListener);
            }
        
            /**
             * Sets footer view state.
             *
             * @param recyclerView  the recycler view
             * @param pageSize      the page size
             * @param state         the state
             * @param errorListener the error listener
             */
            private void setFooterViewState(RecyclerView recyclerView, int pageSize,
                                            BaseEndlessFooterView.State state, View.OnClickListener errorListener) {
                if (mView == null) {
                    Log.e(TAG, "you must set footer view before you use");
                    throw new InvalidParameterException("you must call setFooterView to set footer view before you use");
                }
                // get adapter of recycler view
                RecyclerView.Adapter adapter = recyclerView.getAdapter();
                if (adapter == null) {
                    Log.w(TAG, "setFooterViewState adapter invalidate");
                    return;
                }
                BaseRecyclerAdapter eadapter = (BaseRecyclerAdapter) adapter;
                // if less than one page, not need to add EndlessFooter
                if (eadapter.getBasicItemCount() < pageSize) {
                    return;
                }
        
                // create or get endless footer
                if (eadapter.getFooterView().size() <= 0) { // create a new endless footer view
                    eadapter.addFooterView(mView);
                }
        
                // set state
                mView.setState(state);
        
                // set error listener, if necessary
                if (state == BaseEndlessFooterView.State.Error && errorListener != null) {
                    mView.setOnClickListener(errorListener);
                }
            }
        
            /**
             * get endless footer view status
             *
             * @param recyclerView recycler view
             * @return the footer view state
             */
            public BaseEndlessFooterView.State getFooterViewState(RecyclerView recyclerView) {
                // get adapter of recycler view
                RecyclerView.Adapter adapter = recyclerView.getAdapter();
                if (adapter == null) {
                    Log.w(TAG, "getFooterViewState adapter invalidate");
                    return BaseEndlessFooterView.State.Normal;
                }
                // get endless footer view
                BaseRecyclerAdapter eadapter = (BaseRecyclerAdapter) adapter;
                if (eadapter.getFooterView().size() > 0) {
                    BaseEndlessFooterView endlessFooterView = (BaseEndlessFooterView) eadapter.getFooterView().get(0);
                    return endlessFooterView.getState();
                }
        
                return BaseEndlessFooterView.State.Normal;
            }
        
        }

## 3）现在来看看滚动事件

核心的思路是在onScrolled里面拿到最后一次看的item位置，然后在onScrollStateChanged里面判断是否要调用loadmore方法

    public abstract class HugeRecyclerOnScrollListener extends RecyclerView.OnScrollListener {
        private final static String TAG = HugeRecyclerOnScrollListener.class.getSimpleName();
    
        // all item count by LayoutManager.getItemCount();
        private int mTotalCount;
        // visible item count by LayoutManager.getChildCount();
        private int mVisibleCount;
        // last visible position count by LayoutManager.findLastVisibleItemPosition()
        private int mLastVisiblePosition;
    
        public abstract void onLoadMore();
    
        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();
            mLastVisiblePosition = layoutManager.findLastVisibleItemPosition();
        }
    
        /**
         * newState
         *
         * @param recyclerView the recycler view
         * @param newState     0: 当前屏幕停止滚动；
         *                     1: 屏幕在滚动 且 用户仍在触碰或手指还在屏幕上；
         *                     2: 随用户的操作，屏幕上产生的惯性滑动；
         */
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
    
            // get layout manager
            RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
            mVisibleCount = layoutManager.getChildCount();
            mTotalCount = layoutManager.getItemCount();
    
            // if scroll idle and only one item not visible load more
            // (total count -2 <= last visible position)
            if (mVisibleCount > 0
                    && newState == RecyclerView.SCROLL_STATE_IDLE
                    && mTotalCount - 2 <= mLastVisiblePosition) {
                onLoadMore();
            }
    
            //Log.d(TAG, "mVisibleCount = " + mVisibleCount + ";mTotalCount = " +
            //        mTotalCount + ";mLastVisibleItemPosition = " + mLastVisiblePosition
            //        + ";newState=" + newState);
        }
    }
    
    
## 准备完毕，我们来用吧

我们在Module的build.gradle中引入

    // huge recyclerview library
    compile 'com.open:hugerecyclerview:1.0.2'

下面可以直接使用了：

Activity的Code大部分都类似，不过我增加了mHugeOnScrollListener和OnRefreshSuccess等refresh监听，依据load more的结果来控制footerview的显示

      public class EndlessActivity extends BaseActivity implements IEndlessListener {
        private static final String TAG = "EndlessActivity";
    
        // hint text view
        private TextView mHint;
        // presenter of MVP
        private EndlessPresenter mPresenter;
        // adapter
        private EndlessRecyclerAdapter mAdapter;
        private RecyclerView mRecyclerView;
        private HugeRecyclerOnScrollListener mHugeOnScrollListener;
        protected EndlessFooterUtils mFooterUtil;
    
        // all data account in server
        private static final int TOTAL_SIZE = 64;
        // each page size count
        private static final int PAGE_SIZE = 10;
        // current data for test
        private int mCurrentNum = 0;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
        }
    
        @Override
        protected void initData() {
            mPresenter = new EndlessPresenter();
            mPresenter.setListener(this);
        }
    
        @Override
        protected void initView(Bundle savedInstanceState) {
            setContentView(R.layout.activity_endless);
    
            mRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
            LinearLayoutManager lm = new LinearLayoutManager(this);
            mRecyclerView.setLayoutManager(lm);
            mAdapter = new EndlessRecyclerAdapter(this);
            mRecyclerView.setAdapter(mAdapter);
            mFooterUtil = new EndlessFooterUtils(new EndlessFooterView(this));
            // set scroll listener
            mHugeOnScrollListener = new HugeRecyclerOnScrollListener() {
                @Override
                public void onLoadMore() {
                    EndlessFooterView.State state = mFooterUtil.getFooterViewState(mRecyclerView);
                    // still loading, do nothing
                    if (EndlessFooterView.State.Loading == state) {
                        Log.d(TAG, "still loading, now return");
                        return;
                    }
                    // no more data
                    if (mCurrentNum > TOTAL_SIZE) {
                        mFooterUtil.setEnd(mRecyclerView, PAGE_SIZE);
                        return;
                    }
    
                    refreshData();
                }
            };
            // add scroll listener
            mRecyclerView.addOnScrollListener(mHugeOnScrollListener);
    
            mHint = (TextView) findViewById(R.id.tv_hint);
        }
    
        private void refreshData() {
            if (!NetworkUtils.isConnected(EndlessActivity.this)) {
                mFooterUtil.setError(mRecyclerView, PAGE_SIZE, mFooterClick);
                Log.w(TAG, "net work no connected");
                ToastUtils.INSTANCE.showToast(EndlessActivity.this, "net work no connected",
                        Toast.LENGTH_SHORT);
                return;
            }
    
            // loading more data
            mFooterUtil.setLoading(mRecyclerView, PAGE_SIZE);
            mPresenter.refreshData();
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
        public void OnRefreshSuccess(List<SampleModel> data) {
            mAdapter.addData(data);
            mCurrentNum += data.size();
            mFooterUtil.setNormal(mRecyclerView, PAGE_SIZE);
        }
    
        private View.OnClickListener mFooterClick = new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                refreshData();
            }
        };
    
        @Override
        public void OnRefreshFail(String error) {
            mFooterUtil.setError(mRecyclerView, PAGE_SIZE, mFooterClick);
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            mPresenter.destroy();
        }
    }
    

更多使用方法请查看：[hugerecyclerview](https://bintray.com/vivianwayne1985/maven/hugerecyclerview)    
    
    
## 看看效果图

![](https://i.imgur.com/jl9baMT.jpg)

![](https://i.imgur.com/5jcGgJt.jpg)

![](https://i.imgur.com/1JwjYs6.jpg)

# [RecyclerView封装目录](http://vivianking6855.github.io/2018/02/24/RecyclerView-Advance-index/)
