---
layout: post
title: 性能优化（四）Google典范之Render实践
date: 2017-3-14
excerpt: "Google典范之Render实践"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 前言

优化的思想：尽量减少布局文件的层级和降低Overdraw来减轻CPU和GPU负载。

再贴下CPU和GPU的工作，潜在的问题，检测的工具和解决方案图：
 
![](http://i.imgur.com/SiZVlJ9.png)

# 解决方案

## 1. 通过Hierarchy Viewer去检测渲染效率，去除不必要的嵌套

### 1.1 删除布局中无用的控件和层级

#### 1.1.1 优化前核心Code和Hierarchy Viewer情况

如果找不到Hierarchy Viewer，可以看下面的“如何找到Hierarchy Viewer？”

实例layout




Hierarchy Viewer 分析图示：



点击LinearLayout，然后点击三色Obtain layout time icon. 可以看到两个子View上面都有了3个圈圈

- 取色范围为红、黄、绿色。
- 三个圈圈分别代表measure 、layout、draw的速度，并且你也可以看到实际的运行的速度
- 如果你发现某个View上的圈是红色，那么说明这个View相对其他的View，该操作运行最慢，注意只是相对别的View，并不是说就一定很慢。
- 红色的指示能给你一个判断的依据，具体慢不慢还是需要你自己去判断的。

上面的布局文件展示了两种写法：一个是Linearlayout嵌套的，一个是RelativeLayout搭配StubView和<merge>

从图中来看方案二教快一些（请多次点击Profile Node取样）

Hierarchy Viewer可以帮助我们发现类似的问题。


### 1.2 Layout 设计优化

除了使用Hierarchy Viewer检测无用控件和层级外，在布局设计时如果可以加入下面的优化思想就更好了。

- 有选择地使用性能较低的ViewGroup
- 采用<include>、<merge>标签和ViewStub

## 2. Overdraw方案

   通过Show GPU Overdraw去检测(开发者选项 -> 调试GPU过度绘制 -> 显示GPU过度绘制区域)

   最终可以通过移除不必要的背景以及使用canvas.clipRect解决大多数问题

### 2.1 移除不必要的background

贴下Overdraw颜色说明图。蓝色，淡绿，淡红，深红代表了4种不同程度的Overdraw情况。

我们的目标就是尽量减少红色Overdraw，看到更多的蓝色区域。

![](http://i.imgur.com/BJCf3ps.png)

#### 2.1.1 优化前核心Code和GPU Overdraw情况

----------MainActivity

    public class MainActivity extends AppCompatActivity implements HomeAdapter.OnItemClickLitener {
        private final static String TAG = MainActivity.class.getSimpleName();
    
        private RecyclerView recyclerView;
        private HomeAdapter homeAdapter;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            initView();
        }
    
        private void initView() {
            // init recycler view
            recyclerView = (RecyclerView) findViewById(R.id.homelist);
            homeAdapter = new HomeAdapter(this);
            homeAdapter.setOnItemClickLitener(this);
            recyclerView.setLayoutManager(new LinearLayoutManager(this));
            recyclerView.setAdapter(homeAdapter);
        }
    
        @Override
        public void onItemClick(View view, int position) {
        }
    
        @Override
        public void onItemLongClick(View view, int position) {
    
        }
    }

-------------HomeAdapter

    public class HomeAdapter extends RecyclerView.Adapter {
        public final static String TAG = HomeAdapter.class.getSimpleName();
    
        private ArrayList<Droid> mData;
        private Context mContext;
    
    
        // click listener
        private OnItemClickLitener mOnItemClickLitener;
    
        public HomeAdapter(Context context) {
            mContext = context;
            initData();
        }
    
        protected void initData() {
            mData = (ArrayList) Droid.generateDatas();
        }
    
        @Override
        public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            MyViewHolder holder = new MyViewHolder(LayoutInflater.from(mContext).inflate(R.layout.recycler_item_bad, parent,
                    false));
            return holder;
        }
    
        @Override
        public void onBindViewHolder(final RecyclerView.ViewHolder holder, int position) {
            MyViewHolder viewHolder = (MyViewHolder) holder;
            Droid droid = mData.get(position);
            viewHolder.date.setText(droid.date);
            viewHolder.msg.setText(droid.msg);
            viewHolder.name.setText(droid.name);
    
            viewHolder.icon.setBackgroundColor(Color.parseColor("#F5F5DC"));
            viewHolder.icon.setImageResource(droid.imageId);
        }
    
    
        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position, List payloads) {
            super.onBindViewHolder(holder, position, payloads);
        }
    
        @Override
        public int getItemCount() {
            return mData.size();
        }
    
        // view holder
        class MyViewHolder extends RecyclerView.ViewHolder {
    
            public ImageView icon;
            public TextView name;
            public TextView date;
            public TextView msg;
    
            public MyViewHolder(View view) {
                super(view);
                icon = (ImageView) view.findViewById(R.id.id_chat_icon);
                name = (TextView) view.findViewById(R.id.id_chat_name);
                date = (TextView) view.findViewById(R.id.id_chat_date);
                msg = (TextView) view.findViewById(R.id.id_chat_msg);
            }
        }
    }
    
------------acitivity_main_bad

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white"
        android:orientation="vertical">
    
        <android.support.v7.widget.RecyclerView
            android:id="@+id/homelist"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/white" />
    
    </LinearLayout>

------------recycler_item_bad

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
    
        <ImageView
            android:id="@+id/id_chat_icon"
            android:layout_width="@dimen/avatar_dimen"
            android:layout_height="@dimen/avatar_dimen"
            android:layout_margin="@dimen/avatar_layout_margin" />
    
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@android:color/darker_gray"
            android:orientation="vertical">
    
            <RelativeLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="@android:color/white"
                android:orientation="horizontal"
                android:textColor="#78A">
    
                <TextView xmlns:android="http://schemas.android.com/apk/res/android"
                    android:id="@+id/id_chat_name"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_alignParentLeft="true"
                    android:gravity="bottom"
                    android:padding="@dimen/narrow_space"
                    android:text="@string/hello" />
    
                <TextView xmlns:android="http://schemas.android.com/apk/res/android"
                    android:id="@+id/id_chat_date"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_alignParentRight="true"
                    android:padding="@dimen/narrow_space"
                    android:text="@string/hello"
                    android:textStyle="italic" />
            </RelativeLayout>
    
            <TextView xmlns:android="http://schemas.android.com/apk/res/android"
                android:id="@+id/id_chat_msg"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="@android:color/white"
                android:padding="@dimen/narrow_space"
                android:text="@string/hello" />
        </LinearLayout>
    </LinearLayout>

[完整Code](https://github.com/vivianking6855/android-advanced/tree/master/PerformanceDemo)


GPU Overdraw情况如下图，都是红色: 4X+ overdraw

![](http://i.imgur.com/Ir76PpT.png)


#### 2.1.2 优化背景

- 不必要的Background 1
    
    我们主布局的文件已经是background为white了，那么可以移除RecyclerView的白色背景
    
- 不必要的Background 2
    
    Item布局中的LinearLayout的android:background="@android:color/darker_gray"
    
    它的两个item已经填满整个layout，而且每个item的背景都要求是白色
    
- 不必要的Background 3
    
    Item布局中的RelativeLayout的android:background="@android:color/white"
    
- 不必要的Background 4
    
    Item布局中id为id_msg的TextView的android:background="@android:color/white"
    
- 不必要的Background 5

    Adapter中的onBindViewHolder，每个icon设置了背景色（主要是当没有icon图的时候去显示）

    然后又设置了一个头像。那么就造成了overdraw，有头像的完全没必要去绘制背景
    
        // bad code
        //viewHolder.icon.setBackgroundColor(Color.parseColor("#F5F5DC"));
        //viewHolder.icon.setImageResource(droid.imageId);
        
        // optimized code
        if (droid.imageId == -1) {
            viewHolder.icon.setBackgroundColor(Color.parseColor("#F5F5DC"));
            viewHolder.icon.setImageResource(android.R.color.transparent);
        } else {
            viewHolder.icon.setImageResource(droid.imageId);
            viewHolder.icon.setBackgroundResource(android.R.color.transparent);
        }
        
- 不必要的Background 6

    我们的Activity背景是白色，layout中去设置了背景色白色。

    因为Activity的布局最终会添加在DecorView中，这个DecorView中的背景就没有必要了
    
        setContentView(R.layout.activity_main);
        getWindow().setBackgroundDrawable(null);

#### 2.1.3 优化后效果

是理想的 1X Overdraw

![](http://i.imgur.com/ZTvGmKi.png)


### 2.2 clipRect的妙用

有一些自定义View，例如扑克牌层叠View, 经常会存在很多不必要的绘制。

多张卡片叠加，叠加的区域肯定是过度绘制了。这时候我们可以用clipRect解决这类问题。

#### 2.2.1 优化前核心Code和GPU Overdraw情况

------------------------ClipRectActivity

    public class ClipRectActivity extends AppCompatActivity {
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_clip_rect);
        }
    }

------------------------activity_clip_rect.xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
        <com.vv.performancedemo.view.CardView
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    
    </LinearLayout>

------------------------CardView

    public class CardView extends View {
        private final static String TAG = CardView.class.getSimpleName();
    
        private final static int X_OFFSET = 50; // canvas x offset
        private final static int Y_OFFSET = 100;// canvas y offset
        private final static int IMG_HOR_OFFSET = 200; // image cover start point
    
        private Bitmap[] mCards = new Bitmap[4];
        private int[] mImgId = new int[]{
                R.drawable.card1,
                R.drawable.card2,
                R.drawable.card3,
                R.drawable.card4
        };
    
        public CardView(Context context) {
            this(context, null);
        }
    
        public CardView(Context context, @Nullable AttributeSet attrs) {
            super(context, attrs);
            initView();
        }
    
        private void initView() {
            Bitmap bm = null;
            for (int i = 0; i < mCards.length; i++) {
                bm = ImageUtils.decodeSampledBitmapFromResource(getResources(), mImgId[i], 400, 600);
                mCards[i] = Bitmap.createScaledBitmap(bm, 400, 600, false);
            }
    
            setBackgroundColor(Color.parseColor("#F5F5DC"));
        }
    
        @Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);
    
            canvas.save();
            canvas.translate(X_OFFSET, Y_OFFSET);
    
            for (Bitmap bitmap : mCards) {
                canvas.translate(120, 0);
                canvas.drawBitmap(bitmap, 0, 0, null);
            }
    
            canvas.restore();
        }
    
    }
    
GPU Overdraw情况如下图，都是红色: 4X+ overdraw

![](vivianking6855.github.io/datum/images/cliprect_bad.jpg)

#### 2.2.2 用clipRect优化

逻辑上只有最后一张图需要完整的绘制，其他的都只需要绘制部

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        canvas.save();
        canvas.translate(X_OFFSET, Y_OFFSET);

        for (int i = 0; i < mCards.length; i++) {
            canvas.translate(IMG_HOR_OFFSET, 0);
            canvas.save();
            if (i < mCards.length - 1) {
                canvas.clipRect(0, 0, IMG_HOR_OFFSET, mCards[i].getHeight());
            }
            canvas.drawBitmap(mCards[i], 0, 0, null);
            canvas.restore();
        }

        canvas.restore();
    }


最后不要忘记，我们有给CardView setBackgroundColor，因此DecorView中的背景就没有必要了：

    public class ClipRectActivity extends AppCompatActivity {
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_clip_rect);
    
            getWindow().setBackgroundDrawable(null);
        }
    }

#### 2.2.3 优化后效果

是理想的 1X Overdraw

![](vivianking6855.github.io/datum/images/cliprect_good.jpg)


# AS 三步找到 Hierarchy Viewer

看Android Studio三步找到它

## Step 1：

![](http://i.imgur.com/mCxa2Ow.jpg)

## Step 2：

![](http://i.imgur.com/6GCAp0a.jpg)

## Step 3：

![](http://i.imgur.com/5OHUC4l.jpg)

<br>


另外好需要显示下面三个窗口，不然看不到Tree View，Propertyies, Overview

![](http://i.imgur.com/vpOcJMU.jpg)

<br>

![](http://i.imgur.com/ffC691e.jpg)


# 性能优化专题

- [性能优化（一）：方法概述](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)
- [性能优化（二）内存 OOM](http://vivianking6855.github.io/2017/02/27/Android-optimization-2-OOM/)
- [Android性能优化（三）Google典范之开篇](http://vivianking6855.github.io/2017/03/13/Android-optimization-3-Google-Publish/)
- [性能优化（四）Google典范之Render实践](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)




# Reference

<br>


> [Google 发布 Android 性能优化典范](http://www.oschina.net/news/60157/android-performance-patterns?sid=07vbqo00ovnh233e0ain6ue5a6)

> [优达学城 Android 系统性能 Video](https://cn.udacity.com/course/android-performance--ud825)

> [Google Android Performance Patterns YouTube ](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

> [Android UI性能优化实战 识别绘制中的性能问题](http://blog.csdn.net/lmj623565791/article/details/45556391/)





