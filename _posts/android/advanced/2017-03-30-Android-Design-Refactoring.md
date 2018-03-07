---
layout: post
title: Android Disign And Refactoring 
date: 2017-3-30
excerpt: "Android Disign And Refactoring "
categories: Android
tags: [Android 进阶]
comments: true
---

# 前言

对于一个App来讲，重构一般情况都是必要的。除非一次性把它设计好。

# 什么时候做重构

一般来说，产品的需求是不断增加和变动的。 App的是否需要重构可以从下面几个角度来综合判断。

1. 如果不重构，会导致很严重的功能问题。
    - 必要时可能需要砍掉一些功能
    - 一定要跟planner一起Sync
2. 是否有空余的开发人员来做重构。
    - 可以创建新的分支开单独存放重构code
    - 搭配Q测试验证Pass
    - 分阶段发布（10%, 50%, 100%）
3. 重构计划要事先提出。一次重构最好不超过2W。
    - 如要要重构打的模块或者底层架构，可以考虑分阶段重构。把重构分为几次迭代
    - 分期上线，先重构解决最严重的问题


# 重构

重构的目标：

- 有一个更清晰的项目结构；
- 更好的维护和扩展框架； 
- 相互隔离的业务模块；
- 一些可以复用的公共组件； 

重构的几项实践：

1. 重构规划项目结构
2. 为Activity定义新的生命周期
3. Application处理全局变量，管理Activity等

# 一、 重构规划项目结构

## 1. 主项目

主项目这里列出了两种思路的项目结构. 项目采用MVP架构

### 1） 全部按照模块来划分

- 各个独立业务模块：每个模块中都包含自己的activity, adapter, model（例如splash，UserInfo等）
- db： sqllite相关逻辑封装
- ui: 所有自定义控件
- utils： 所有公用方法
- Interfaces： 真正意义上的接口，命名以I作为开头
- listener： 基于Listener的接口，命名以On作为开头
- presenter：所有数据业务逻辑在这里
- model：数据模型，数据api，不直接访问View

### 2） Activity按照模块划分

- activity: 按照模块划分，把不同Activity放到不同的模块包下面
- adapter: 所有的适配器
- model: 所有的实体
- db： sqllite相关逻辑封装
- ui: 所有自定义控件
- utils： 所有公用方法
- Interfaces： 真正意义上的接口，命名以I作为开头
- listener： 基于Listener的接口，命名以On作为开头
- presenter：所有数据业务逻辑在这里
- model：数据模型，数据api，不直接访问View

这里Activity按照模块拆分了，adapter和entity没有拆分。这是因为：

- 我们看Code的时候通常是从Activity看起。
- 此外Adapter逻辑大同小异。Entity中应该只有属性，如果确实需要，请把它已到engine包里面（当Entity有上百个时，我们可以考虑按照模块拆分）
- 每个Activity都有着很复杂的业务逻辑，所以Activity才是最重要的。

## 2. 类库

不管是 1）还是2）都建议建立一个类库AndroidLib，将业务无关的逻辑转移到这里。

AndroidLib程序包有：（全部都是业务无关）

- com.open.utislib.base	 
- com.open.utislib.device	 
- com.open.utislib.file	 
- com.open.utislib.net	 
- com.open.utislib.security	 
- com.open.utislib.time	 
- com.open.utislib.window

已发布到JCenter：[utilslib](https://bintray.com/vivianwayne1985/maven/utilslib)

### 小结

- 重构规划项目结构有的好处：
    - 每个文件只有一个单独的类，不要有嵌套类
    - 将Activity按照模块拆分后，可以迅速定位具体的页面
    - 类库方便重用
- 不建议在Activity中嵌套adapter,entity. 
- 个人比较喜欢项目结构1），因为移除某个功能非常的便捷。

这里仅列出两个示例的项目结构，大家可以依照自己的需求定制合适的项目结构。

# 二、 为Activity定义新的生命周期

优化onCreate里面的可读性，分为三个part来各自做一件事情

- initData() 初始化数据
- initView() 初始化控件
- loadData() 加载数据
- getLayout() 设置layout

    public abstract class BaseActivity extends AppCompatActivity {
    
        @Override
        protected void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(getLayout());
    
            initData();
            initView(savedInstanceState);
            loadData();
        }
    
        /**
         * Init data.
         */
        protected abstract void initData();
    
        /**
         * Init view.
         *
         * @param savedInstanceState the saved instance state
         */
        protected abstract void initView(Bundle savedInstanceState);
    
        /**
         * Load data.
         */
        protected abstract void loadData();
        
        
# 三、 重写Application

重写Application亦可以对项目性能，重构带来非常好的影响。（不过建议在项目创建时，就先考虑到，后面重构时可以不断完善）

重写Application有下面的用处：

1. 保存全局变量，整个应用内共享数据 
2. 管理Activity。应用退出时，销毁所有的Activity 
3. 初始化应用程序的配置信息 
4. 系统内存不足时，做出合理的响应 

继承Application，重写之后，要在AndroidManifest.xml中配置Application节点的name属性，比如

    <application
        android:name=".UserApplication"
        ......>
        
重写的Application类：

    /**
     * Created by vivian on 2017/11/13.
     * User customized Application
     */
    public class UserApplication extends Application {
        private UserApplication mInstance;
    
        /**
         * single instance
         *
         * @return instance
         */
        public UserApplication getInstance() {
            if (mInstance == null) {
                return new UserApplication();
            }
            return mInstance;
        }
    
        /**
         * 在创建应用程序时调用，可以在这里实例化应用程序的单例，
         * 创建和实例化任何应用程序状态变量或共享资源等
         */
        @Override
        public void onCreate() {
            super.onCreate();
    
        }
    
        /**
         * 程序终止时调用，如果需要及时的释放资源可以在各个Activity中。
         * onTerminate只有在程序终止时才会调用，程序退到后台时有可能不会终止。
         */
        @Override
        public void onTerminate() {
            super.onTerminate();
        }
    
        /**
         * 当系统处于资源匮乏时，具有良好行为的应用程序可以释放额外的内存。
         * 这个方法一般只会在后台进程已经终止，但是前台应用程序仍然缺少内存时调用。
         * 我们可以重写这个程序来清空缓存或者释放不必要的资源
         */
        @Override
        public void onLowMemory() {
            super.onLowMemory();
        }
    
        /**
         * 作为onLowMemory的一个特定于应用程序的替代选择，在android4.0时引入，
         * 在程序运行时决定当前应用程序应该尝试减少其内存开销时（通常在它进入后台时）调用
         * 它包含一个level参数，用于提供请求的上下文
         * level 的值: ComponentCallbacks2.TRIM_MEMORY_...
         */
        @Override
        public void onTrimMemory(int level) {
            super.onTrimMemory(level);
        }
    
    
        /**
         * 在配置改变时，应用程序对象不会被终止和重启。
         * 如果应用程序使用的值依赖于特定的配置，则重写这个方法来重新加载这些值，或者在应用程序级别处理这些值的改变
         */
        @Override
        public void onConfigurationChanged(Configuration newConfig) {
            super.onConfigurationChanged(newConfig);
        }
    
    }



# MVP 项目结构

这里贴出自己使用的基于MVP架构的项目结构


![](https://i.imgur.com/zbW9vDL.png)


github[下载地址](https://github.com/vivianking6855/android-advanced/tree/master/AppUniform)

# Reference

> 《App研发录》

> [App研发录作者 Blog](http://www.cnblogs.com/jax/p/4656789.html)

> [如何打造 最合适的构架，最合适的重构？](http://mp.weixin.qq.com/s/Uj4zbErp2_5RjUsKlkYiZA)