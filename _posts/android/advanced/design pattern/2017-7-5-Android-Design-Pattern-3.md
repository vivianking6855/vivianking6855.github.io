---
layout: post
title: Android Design-Pattern （三）
date: 2017-7-5
excerpt: "Android Disign Pattern （三）"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 行为型模式11种

## 13. 观察者模式（Observer Pattern）-- Android常用模式

### 简介

一个对象发生改变时，所有信赖于它的对象自动做相应改变。

例如下面的订阅感兴趣事物场景

- 天气预报短信服务，一旦你订阅该服务，你只需按月付费，付完费后，每天一旦有天气信息更新，它就会及时向你发送最新的天气信息。
- 杂志的订阅，你只需向邮局订阅杂志，缴纳一定的费用，当有新的杂志时，邮局会自动将杂志送至你预留的地址。

观察者模式的几个重要组成。

- 观察者，我们称它为Observer，有时候我们也称它为订阅者，即Subscriber
- 被观察者，我们称它为Observable，即可以被观察的东西，有时候还会称之为主题，即Subject

### 核心Code 

java中提供了Observable类和Observer接口供我们快速的实现该模式

----------------被观察者

    public class House extends Observable {
        private float price;// 价钱
    
        public House(float price) {
            this.price = price;
        }
    
        public float getPrice() {
            return this.price;
        }
    
        public void setPrice(float price) {
            // 每一次修改的时候都应该引起观察者的注意
            super.setChanged();    // 设置变化点
            super.notifyObservers(price);// 价格被改变
            this.price = price;
        }
    
        public String toString() {
            return "房子价格为：" + this.price;
        }
    }

--------------观察者

    public class HousePriceObserver implements Observer {
        private String name;
    
        public HousePriceObserver(String name) { // 设置每一个购房者的名字
            this.name = name;
        }
    
        @Override
        public void update(Observable o, Object arg) {
            if (arg instanceof Float) {
                System.out.print(this.name + "观察到价格更改为：");
                System.out.println(((Float) arg).floatValue());
            }
        }
    
    }

--------------实际使用

    public class HouseDemo {
        public void start() {
            House h = new House(1000000);
            HousePriceObserver hpo1 = new HousePriceObserver("购房者A");
            HousePriceObserver hpo2 = new HousePriceObserver("购房者B");
            HousePriceObserver hpo3 = new HousePriceObserver("购房者C");
            h.addObserver(hpo1);
            h.addObserver(hpo2);
            h.addObserver(hpo3);
            System.out.println(h); // 输出房子价格
            h.setPrice(666666);    // 修改房子价格
            System.out.println(h); // 输出房子价格
        }
    }


### 已有示例

Android的事件通知就是典型的观察者模式

    Button btn=new Button(this);
    btn.setOnClickListener(new View.OnClickListener() {
    	@Override
    	public void onClick(View v) {
    		Log.e("TAG","click");
    	}
    })


Android的广播机制，其本质也是观察者模式。例如LocalBroadcastManager。

我们平时使用本地广播主要就是下面四个方法

    LocalBroadcastManager localBroadcastManager=LocalBroadcastManager.getInstance(this);
    localBroadcastManager.registerReceiver(BroadcastReceiver receiver, IntentFilter filter);
    localBroadcastManager.unregisterReceiver(BroadcastReceiver receiver);
    localBroadcastManager.sendBroadcast(Intent intent)

调用registerReceiver方法注册广播，调用unregisterReceiver方法取消注册，之后直接使用sendBroadcast发送广播

发送广播之后，注册的广播会收到对应的广播信息，这就是典型的观察者模式。

开源库EventBus也是基于观察者模式的，观察者模式的三个典型方法它都具有，即注册，取消注册，发送事件

    EventBus.getDefault().register(Object subscriber);
    EventBus.getDefault().unregister(Object subscriber);
    EventBus.getDefault().post(Object event);

重量级的并发库RxJava也是观察者模式

    创建一个被观察者
    
    Observable<String> myObservable = Observable.create(  
        new Observable.OnSubscribe<String>() {  
            @Override  
            public void call(Subscriber<? super String> sub) {  
                sub.onNext("Hello, world!");  
                sub.onCompleted();  
            }  
        }  
    );
    
    创建一个观察者，也就是订阅者
    
    Subscriber<String> mySubscriber = new Subscriber<String>() {  
        @Override  
        public void onNext(String s) { System.out.println(s); }  
      
        @Override  
        public void onCompleted() { }  
      
        @Override  
        public void onError(Throwable e) { }  
    };
    
    观察者进行事件的订阅
    
    myObservable.subscribe(mySubscriber);

总之，在Android中观察者模式还是被用得很频繁的。

## 14. 中介者模式（Mediator Pattern）

### 简介

用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

### 核心Code 

### 已有示例

Binder机制

## 15. 访问者模式（Visitor Pattern）

### 简介

表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

### 核心Code 

### 已有示例

编译时注解中的ElementVisitor中定义多个Visit接口，每个接口处理一种数据类型，这就是典型的访问者模式

访问者模式正好解决了数据结构和数据操作分离的问题，避免某些操作污染了数据对象类。

## 16. 解释器模式（Interpreter Pattern）

### 简介

给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。

### 核心Code

PackageParser这个类对AndroidManifest.xml这个配置文件的解析过程，

### 已有示例

## 17. 迭代器模式（Iterator Pattern）

### 简介

提供一种方法顺序访问一个聚合对象中各个元素, 而又不需暴露该对象的内部表示。

### 核心Code 

### 已有示例

在Android中除了各种数据结构体，如List，Map，等包含的迭代器以外，Android源码中也提供了迭代器遍历模式

比如数据库查询使用Cursor，当我们使用SQLiteDataBase的query方法查询数据库时，会返回一个Cursor游标对象，该游标对象实际上就是一个具体的迭代器。

## 18. 备忘录模式（Memento Pattern）

### 简介

不需要了解对象的内部结构的情况下备份对象的状态，方便以后恢复。

### 核心Code 

### 已有示例

Activity的onSaveInstanceState和onRestoreInstanceState就是通过Bundle这种序列化的数据结构来存储Activity的状态，至于其中存储的数据结构，这两个方法不用关心。

## 19. 责任链模式（Chain of Responsibility Pattern）

### 简介

有多个的对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定。

### 核心Code 

### 已有示例

责任链模式在Android源码中比较类似的实现莫过于对事件的分发处理

每当用户接触屏幕时候，Android都会将对应的事件包装成一个事件对象从ViewTree的顶部至上而下的分发传递。

ViewGroup事件投递的递归调用就类似一条责任链，一旦寻找到责任者，那么就由责任者持有并消费该次事件

具体的体现在View的onTouchEvent方法中的返回值，如果OnTouchEvent返回false，那么意味着当前View不会是该次事件的责任人，将不会对该事件持有。

## 20. 状态模式（State Pattern）

### 简介

状态发生改变时，行为改变。

### 核心Code 

### 已有示例

View.onVisibilityChanged方法，就是提供了一个状态模式的实现，允许在View的visibility发生改变时，引发执行onVisibilityChanged方法中的动作。

## 21. 策略模式（） -- Android常用模式

### 简介

定义了一系列封装了算法、行为的对象，他们可以相互替换。

策略模式让算法独立于使用它的客户而独立变换。


### 核心Code

假设我们要出去旅游，而去旅游出行的方式有很多，有步行，有坐火车，有坐飞机等等。

不使用任何模式有一个致命的缺点，一旦出行的方式要增加，我们就不得不增加新的else if语句，而这违反了面向对象的原则之一，对修改封闭。

策略模式则可以完美的解决这一切。应用了策略模式后，如果我们想增加新的出行方式，完全不必要修改现有的类，我们只需要实现策略接口即可，这就是面向对象中的对扩展开放准则。

假设现在我们增加了一种自行车出行的方式。只需新增一个类即可。

    -------------------不使用任何模式


    public class TravelStrategy {
        enum Strategy {
            WALK, PLANE, SUBWAY
        }
    
        private Strategy strategy;
    
        public TravelStrategy(Strategy strategy) {
            this.strategy = strategy;
        }
    
        public void travel() {
            if (strategy == Strategy.WALK) {
                print("walk");
            } else if (strategy == Strategy.PLANE) {
                print("plane");
            } else if (strategy == Strategy.SUBWAY) {
                print("subway");
            }
        }
    
        public void print(String str) {
            System.out.println("出行旅游的方式为：" + str);
        }
    
        public static void main(String[] args) {
            TravelStrategy walk = new TravelStrategy(Strategy.WALK);
            walk.travel();
    
            TravelStrategy plane = new TravelStrategy(Strategy.PLANE);
            plane.travel();
    
            TravelStrategy subway = new TravelStrategy(Strategy.SUBWAY);
            subway.travel();
        }
    }

    -------------策略模式

    ---接口

    public interface IStrategy {
        void travel();
    }
    
    ---各种travel方式类
    
    public class PlaneStrategy implements IStrategy {
        @Override
        public void travel() {
            System.out.println("plane");
        }
    }
    
    public class SubwayStrategy implements IStrategy {
        @Override
        public void travel() {
            System.out.println("subway");
        }
    }
    
    public class WalkStrategy implements IStrategy {
        @Override
        public void travel() {
            System.out.println("walk");
        }
    }
    
    ---包装策略的类
    
    public class TravelContext {
        IStrategy strategy;
    
        public IStrategy getStrategy() {
            return strategy;
        }
    
        public void setStrategy(IStrategy strategy) {
            this.strategy = strategy;
        }
    
        public void travel() {
            if (strategy != null) {
                strategy.travel();
            }
        }
    }
    
    ---实际使用
    
    public class TravalStrategyNew {
        public static void main(String[] args) {
            TravelContext travelContext = new TravelContext();
            travelContext.setStrategy(new PlaneStrategy());
            travelContext.travel();
            travelContext.setStrategy(new WalkStrategy());
            travelContext.travel();
            travelContext.setStrategy(new SubwayStrategy());
            travelContext.travel();
        }
    }


### 已有示例

Java.util.List

- Java.util.List就是定义了一个增（add）、删（remove）、改（set）、查（indexOf）策略
- 至于实现这个策略的ArrayList、LinkedList等类，只是在具体实现时采用了不同的算法。
- 它们策略一样，不考虑速度的情况下，使用时完全可以互相替换使用。

属性动画中插值器TimeInterpolator 

- 它的作用就是根据时间流逝的百分比来来计算出当前属性值改变的百分比
- 使用属性动画的时候,可以通过set方法对插值器进行设置.可以看到内部维持了一个时间插值器的引用，并设置了getter和setter方法，默认情况下是先加速后减速的插值器
    
    set方法如果传入的是null，则是线性插值器
    
- 而时间插值器TimeInterpolator是个接口，有一个接口继承了该接口，就是Interpolator这个接口，其作用是为了保持兼容

    private static final TimeInterpolator sDefaultInterpolator =
    		new AccelerateDecelerateInterpolator();  
    		
    private TimeInterpolator mInterpolator = sDefaultInterpolator; 
    
    @Override
    public void setInterpolator(TimeInterpolator value) {
    	if (value != null) {
    		mInterpolator = value;
    	} else {
    		mInterpolator = new LinearInterpolator();
    	}
    }

    @Override
    public TimeInterpolator getInterpolator() {
    	return mInterpolator;
    }
    
    public interface Interpolator extends TimeInterpolator {
        // A new interface, TimeInterpolator, was introduced for the new android.animation
        // package. This older Interpolator interface extends TimeInterpolator so that users of
        // the new Animator-based animations can use either the old Interpolator implementations or
        // new classes that implement TimeInterpolator directly.
    }
    
- 还有一个BaseInterpolator插值器实现了Interpolator接口，并且是一个抽象类

    abstract public class BaseInterpolator implements Interpolator {
        private int mChangingConfiguration;
        /**
         * @hide
         */
        public int getChangingConfiguration() {
            return mChangingConfiguration;
        }
    
        /**
         * @hide
         */
        void setChangingConfiguration(int changingConfiguration) {
            mChangingConfiguration = changingConfiguration;
        }
    }
 
- 平时我们使用的时候，通过设置不同的插值器，实现不同的动画速率变换效果，比如线性变换，回弹，自由落体等等。这些都是插值器接口的具体实现，也就是具体的插值器策略。 


## 22. 命令模式（Command Pattern）

### 简介

把请求封装成一个对象发送出去，方便定制、排队、取消。

### 核心Code 

### 已有示例

示例：Handler.post后Handler.handleMessage。

## 23. 模板模式（Template Method Pattern）

### 简介

### 核心Code 

### 已有示例


# 设计模式专题

- [设计模式专题一：创建型模式5种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-1/) 
- [设计模式专题二：结构型模式7种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-2/) 
- [设计模式专题二：行为型模式11种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-3/) 

Github Code: [https://github.com/vivianking6855/android-advanced/tree/master/DesignPattern](https://github.com/vivianking6855/android-advanced/tree/master/DesignPattern)

# Reference

[Android开发中常见的设计模式](http://www.cnblogs.com/android-blogs/p/5530239.html)

[Android设计模式之23种设计模式一览](http://blog.csdn.net/happy_horse/article/details/50908439)

[《Android深入透析》之常用设计模式经验谈](https://my.oschina.net/u/2249934/blog/343441)

《android之大话设计模式》

[设计模式中英文对照](https://wenku.baidu.com/view/a216bfa651e79b896802269a.html)

[Android 设计模式](http://blog.csdn.net/banketree/article/details/24985607)