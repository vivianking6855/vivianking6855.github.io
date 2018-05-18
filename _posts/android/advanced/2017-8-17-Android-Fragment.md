---
layout: post
title: Android Fragment
date: 2017-8-17
excerpt: "Android Fragment"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

Fragment可以当成Activity的一个界面的一个组成部分，Activity的界面可以完全有不同的Fragment组成

Fragment拥有自己的生命周期和接收、处理用户的事件。可以动态的添加、替换和移除某个Fragment。

# 1. Fragments and UI Modularization

Fragments实现UI 模块化布局

activity_main.xml
    
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <!-- List of Book Titles -->
        <fragment
            android:id="@+id/fragmentTitles"
            android:name="com.open.dynamicfragments.BookListFragment"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />
    
        <!-- Description of selected book -->
        <fragment
            android:id="@+id/fragmentDescription"
            android:name="com.open.dynamicfragments.BookDescFragment"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />
    </LinearLayout>


fragment_book_desc.xml

    <?xml version="1.0" encoding="utf-8"?>
    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/scrollDescription"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="fill_horizontal"
            android:text="@string/dynamicUiDescription" />
    </ScrollView>


fragment_book_list.xml
    
    <?xml version="1.0" encoding="utf-8"?>
    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/scrollTitles"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
        <RadioGroup
            android:id="@+id/bookSelectGroup"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
    
            <RadioButton
                android:id="@+id/dynamicUiBook"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:checked="true"
                android:text="@string/dynamicUiTitle" />
    
            <RadioButton
                android:id="@+id/android4NewBook"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/android4NewTitle" />
            <!-- Other RadioButtons elided for clarify -->
        </RadioGroup>
    </ScrollView>

[完整Code](https://github.com/vivianking6855/android-advanced/tree/master/DynamicFragments/DynamicFragments-Chapter1)

# 2. Fragments and UI Flexibility

## Fragment实现动态布局

功能需求

- 横屏选择书单，右侧内容变化
- 竖屏选择书单，开启另外一个Activity

实现逻辑：

- 横屏main layout包含两个Fragment（BookDescFragment，BookListFragment）在MainActivity中
- 竖屏main layout一个Fragment（BookListFragment）在MainActivity中；BookDescFragment在另外一个BookDescActivity
- 判断逻辑：如果BookDescFragment存在main layout中，则更新右侧内容，否则叫起另一个Activity

values-land家中使用refs.xml layout alias方式实现

    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <item name="activity_main" type="layout">
            @layout/activity_main_wide
        </item>
    </resources>


## Fragment间交互

- BookListFragment定义Listener
- MainActivity中监听，然后调用BookDescFragment中的方法。

[完整Code](https://github.com/vivianking6855/android-advanced/tree/master/DynamicFragments/DynamicFragments-Chapter2)

# 3. 生命周期

[Activity 生命周期](https://developer.android.com/reference/android/app/Activity.html#ActivityLifecycle)

Fragment 生命周期

![](http://i.imgur.com/h61e5oE.jpg)

![](http://i.imgur.com/PqQfmUA.jpg)


# 4. FragmentTransaction

FragmentTransaction 实现Activity中Fragment的灵活管理和切换。

Demo是在Demo 3的基础上进行优化，竖屏改为一个Activity，动态改变现实不同Fragment（bookList和bookDescription)

activity_mian.xml变化：移除fragmentDescription，后续动态替换

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/layoutRoot"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
    
        <!-- List of Book Titles ** using the ListFragment **-->
        <fragment
            android:id="@+id/fragmentTitles"
            android:name="com.open.dynamicfragments.BookListFragment"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />
    
    </LinearLayout>


MainActivity变化：onCreate加入判断，是否动态支援显示；onSelectedBookChanged 动态替换Fragment

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FragmentManager fm = getSupportFragmentManager();
        Fragment bookDescFragment =
                fm.findFragmentById(R.id.fragmentDescription);
        // If not found than we're doing dynamic mgmt
        mIsDynamic = bookDescFragment == null || !bookDescFragment.isInLayout();

        // Load the list fragment if necessary
        if (mIsDynamic) {
            // Begin transaction
            FragmentTransaction ft = fm.beginTransaction();
            // Create the Fragment and add
            BookListFragment listFragment = new BookListFragment();
            ft.add(R.id.layoutRoot, listFragment, "bookList");
            ft.commit();
        }
    }
    
    
     @Override
    public void onSelectedBookChanged(int bookIndex) {
        BookDescFragment bookDescFragment;
        FragmentManager fm = getSupportFragmentManager();

        // Check validity of fragment reference
        if (mIsDynamic) {
            // Handle dynamic switch to description fragment
            FragmentTransaction ft = fm.beginTransaction();
            bookDescFragment = new BookDescFragment();

            // Create the fragment and attach book index
            bookDescFragment = new BookDescFragment();
            Bundle args = new Bundle();
            args.putInt(BookDescFragment.BOOK_INDEX, bookIndex);
            bookDescFragment.setArguments(args);

            // Replace the book list with the description
            ft.replace(R.id.layoutRoot,
                    bookDescFragment, "bookDescription");
            ft.addToBackStack(null);

            //ft.setCustomAnimations(android.R.animator.fade_in, android.R.animator.fade_out);
            ft.commit();
        } else {
            // Use the already visible description fragment
            bookDescFragment = (BookDescFragment)
                    fm.findFragmentById(R.id.fragmentDescription);
            bookDescFragment.setBook(bookIndex);
        }
    }


BookDescFragment变化： onCreateView加入getArguments处理

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View viewHierarchy = inflater.inflate(R.layout.fragment_book_desc, container, false);

        initView(viewHierarchy);

        return viewHierarchy;
    }

    private void initView(View viewHierarchy) {
        // Load array of book descriptions
        mBookDescriptions = getResources().getStringArray(R.array.bookDescriptions);
        // Get reference to book description text view
        mBookDescriptionTextView = (TextView)
                viewHierarchy.findViewById(R.id.bookDescription);

        // Retrieve the book index if attached
        Bundle args = getArguments();
        int bookIndex = args != null ?
                args.getInt(BOOK_INDEX, BOOK_INDEX_NOT_SET) :
                BOOK_INDEX_NOT_SET;

        // If we find the book index, use it
        if (bookIndex != BOOK_INDEX_NOT_SET) {
            setBook(bookIndex);
        }
    }


# 5. Rich Navigation

Swipe navigation and Action Bar navigation, you could check android default template here:

![](http://i.imgur.com/rXoj18B.jpg)

# 6. Fragement的交互

Fragment可以通过getActivity获取FragmentActivity的实例，也可以通过findFragmentById的方式获取Fragment。

因此Fragment之间的交互有很多种：

- 1) Fragment之间通过Activity通讯
	- 使用方法：Fragment可以通过getActivity
	- 缺点：Fragment要持有Activity的实例
	- 优点：避开了多个Fragment之间的耦合
- 2）Fragment之间直接通讯
	- 使用方法：findFragmentById的方式获取Fragment
	- 缺点：多个Fragment直接耦合
	- 优点：不通过Activity传递
- 3）使用EventBus解耦
	- 使用方法：[参照EventBus使用方法](http://greenrobot.org/eventbus/documentation/how-to-get-started/)
	- 优点：实现Activity，Fragment直接解耦，可定制线程环境
	- 缺点：需要增加EventBus类，逻辑复杂时可能出现nested Event
- 4) RxJava
	- 使用方法：[放弃RxBus，拥抱RxJava](https://www.jianshu.com/p/61631134498e)，透出Observe对象
	- 优点：不通过第三方传递，直接投给目标，可定制线程环境，面向响应式编程
	- 缺点：Fragment之间耦合
- 5）LocalBroadcast方式
	- 使用方法：参考LocalBroadcast[源码解析](https://www.jianshu.com/p/6f1ea19a17ed)和[使用](https://blog.csdn.net/jarchie520/article/details/71434833) 
	- 优点：解除耦合，较EventBus性能差不太多
	- 缺点：代码量多，依赖Context注入，运行在主线程，若有耗时操作可能会影响UX

[EventBus和LocalBroadcast的比较](https://blog.csdn.net/sanjay_f/article/details/50342057)

# Reference

> [Creating Dynamic UI with Android Fragments](https://github.com/vivianking6855/android-advanced/blob/master/DynamicFragments/Creating%20Dynamic%20UI%20with%20Android%20Fragments.pdf)

> [Android 基础：Fragment，看这篇就够了 （上）](https://cloud.tencent.com/developer/article/1005538)

> [Activity与Fragment的生命周期](http://blog.csdn.net/chun0801/article/details/51956847) 
