---
layout: post
title: Android Design-Pattern （一）
date: 2017-7-3
excerpt: "Android Disign Pattern （一）"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}




# 前言

android开发中，必要的了解一些设计模式又是非常有必要的。 Android开发的设计模式，基本设计思想源于java的设计模式

java的设计模式有N多种，据不完全统计，迄今为止，网络出现最频繁的大概有23种。

设计模式的出现就是为了高质量、易维护和复用性强的代码

# 什么是设计模式？

- 基本定义：设计模式（Design pattern）是一套被反复使用的代码设计经验的总结。
    - 使用设计模式的目的是为了可重用代码、让代码更容易被他人理解。
    - 设计模式是是软件工程的基石脉络，如大厦的结构一样。
- Design pattern的四大要素：模式名称（Name），问题（Question），解决方案（Solution），效果（Efftive）。
- OO（面向对象）的六大原则：单一职责原则，开闭原则，里氏替换原则，依赖倒置原则，接口隔离原则，迪米特原则。
    - 单一职责原则：一个类中应该是一组相关性很高的函数，数据的封装。两个完全不一样的功能就不应该放在一个类中。
    - 开闭原则：对修改封闭，对扩展放开。
    - 里氏替换原则：抽象和继承；所有引用基类的地方必须能透明的使用其子类的对象。
    - 依赖倒置原则：抽象不应该依赖细节，细节应该依赖抽象。
    - 接口隔离原则：将大接口改成多个小接口。
    - 迪米特原则：也称为最少知识原则，一个对象应该对另一个对象有最少的了解。

# 设计模式的分类

设计模式分为三种类型：

1. 创建型模式5种：单例模式，抽象工厂模式，工厂模式，原型模式，建造者模式。
2. 结构型模式7种：适配器模式，桥接模式，装饰模式，组合模式，外观模式，享元模式，代理模式。
3. 行为型模式11种：观察者模式，中介者模式，访问者模式，解释器模式，迭代器模式，备忘录模式，责任链模式，状态模式，策略模式，命令模式，模板模式。

# 创建型模式5种

## 1. 单例模式（Singleton Pattern）-- Android常用模式

### 简介

保证一个类仅有一个实例，全局只有一个访问点。

### 核心Code 

实现方法详解看[这里](http://www.jianshu.com/p/eb30a388c5fc)

    public class Singleton {
        private static class SingletonHolder {
            private static final Singleton INSTANCE = new Singleton();
        }
    
        private Singleton() {
        }
    
        public static final Singleton getInstance() {
            return SingletonHolder.INSTANCE;
        }
    }
    
### 已有示例

Android中的系统级服务都是通过容器的单例模式实现方式，以单例形式存在，减少了资源消耗。


## 2. 工厂模式（Factory Pattern）

### 简介

定义一个用于创建对象的接口，让子类决定将哪一个类实例化。

### 核心Code 

麦当劳的点餐，可以点可乐，汉堡这可以使用Builder模式，也可以点套餐，这个可以认为是工厂模式

    public class FactoryDemo {
        public static void start() {
            Order order = OrderFactory.createBigMacCombo();
            System.out.println(order.makeOrder());
        }
    }
    
    ------------------------Factory
    
    public class OrderFactory {
        //创建一份巨无霸套餐
        public static Order createBigMacCombo() {
            return new Order.OrderBuilder()
                    .addBurger(new BigMac())
                    .addBeverage(new Coke())
                    .build();
        }
    }
    
    --------------------------Order
    
    public class Order {
        private IBurgers mBurger;
        private IBeverages mBeverages;
    
        private Order(OrderBuilder builder){
            mBurger = builder. mBurger;
            mBeverages = builder. mBeverages;
        }
    
    
        public String makeOrder(){
            StringBuilder sb = new StringBuilder();
            if ( mBurger!= null) {
                sb.append( mBurger.makeBurger()).append( " ");
            }
            if ( mBeverages!= null) {
                sb.append( mBeverages.makeDrinking()).append( " ");
            }
            return sb.toString();
        }
    
        public static class OrderBuilder{
            private IBurgers mBurger;
            private IBeverages mBeverages;
            public OrderBuilder(){
    
            }
            public OrderBuilder addBurger(IBurgers burgers){
                this. mBurger = burgers;
                return this;
            }
            public OrderBuilder addBeverage(IBeverages beverages){
                this. mBeverages = beverages;
                return this;
            }
    
            public Order build(){
                return new Order( this);
            }
        }
    }
    
    ---------------------------------Product
    
    public class BigMac implements IBurgers {
    
        @Override
        public String makeBurger() {
            return "巨无霸";
        }
    
    }
    
    public class Coke implements IBeverages {
    
        @Override
        public String makeDrinking() {
            return "可乐";
        }
    
    }
    
    public interface IBeverages {
        String makeDrinking();
    }
    
    public interface IBurgers {
        String makeBurger();
    }

### 已有示例

Android中，BitmapFactory用于从不同的数据源来解析、创建Bitmap对象

## 3. 抽象工厂模式（Abstract Factory Pattern）

### 简介

提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

一个用于创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类的实例化延迟到其子类。

使用场景：一个对象族或者一组没有任何关系的对象都有相同的约束，都可以使用抽象工厂模式

- 当调用者需要一个产品时，直接传递一个参数给工厂，让工厂生产不同的产品。
- 这些产品实现了同样的接口。
- 调用者无需了解细节，只需要提要求（传参数）给工厂即可。

角色介绍：

- AbstractProduct: 抽象产品类
- ConcreteProductA : 产品的具体实现A
- ConcreteProductB : 产品的具体实现B
- AbstractFactory : 抽象工厂
- ConcreteFactory : 具体工厂实现

但是抽象工厂模式有个最大的缺点：产品族扩展非常困难，严重违反了开闭原则

### 核心Code 

用户只需要告诉Factory要的产品，不需要关心产品是如何生产的。不过每增加一样产品，BaseAppFactory都要增加一个方法，然后所有的实现类都要修改


    public class FactoryDemo {
        public static void start() {
            BaseAppFactory factory = new MacAppFactory();
            BaseTextEditor textEditor = factory.createTextEditor();
            textEditor.edit();
            textEditor.save();
    
            BaseImageEditor imageEditor = factory.createImageEditor();
            imageEditor.edit();
            imageEditor.save();
        }
    }
    
    --------------------Factory
    
    
    public class MacAppFactory extends BaseAppFactory {
        @Override
        public BaseTextEditor createTextEditor() {
            return new MacTextEditor();
        }
    
        @Override
        public BaseImageEditor createImageEditor() {
            return new MacImageEditor();
        }
    }
    
    public abstract class BaseAppFactory {
        public abstract BaseTextEditor createTextEditor();
    
        public abstract BaseImageEditor createImageEditor();
    }
    
    -----------------Products
    
    public class MacImageEditor extends BaseEditor {
        @Override
        public void edit() {
            System.out.println("图片处理编辑器,edit -- Mac版");
        }
    
        @Override
        public void save() {
            System.out.println("图片处理编辑器,save -- Mac版");
        }
    }
    
    public class MacTextEditor extends BaseEditor {
        @Override
        public void edit() {
            System.out.println("文本编辑器,edit -- Mac版");
        }
    
        @Override
        public void save() {
            System.out.println("文本编辑器,edit -- Mac版");
        }
    }
    
    public abstract class BaseEditor {
        public abstract void edit();
    
        public abstract void save();
    }


### 已有示例

Android底层对MediaPlayer的创建。

MediaPlayerFactory是Android底层为了创建不同的MediaPlayer所定义的一个类。

## 4. 原型模式（Prototype Pattern）-- Android常用模式

### 简介

用原型实例指定创建对象的种类，并且通过拷贝这个原型来创建新的对象。基本可以理解成实现了clone方法

使用场景：在系统中要创建大量的对象，这些对象之间具有几乎完全相同的功能，只是在细节上有一点儿差别

- 比如我们需要一张
- 的几种不同格式：ARGB_8888、RGB_565、ARGB_4444、ALAPHA_8等。
    
    那我们就可以先创建一个ARGB_8888的Bitmap作为原型，在它的基础上，通过调用Bitmap.copy(Config)来创建出其它几种格式的Bitmap。
    
- 另外一个例子就是Java中所有对象都有的一个名字叫clone的方法，已经原型模式的代名词了。

### 核心Code 

核心Code

    注意像ListView这类的需要深度copy
    
    public class Person implements Cloneable {
        private String name;
        private int age;
        private double height;
        private double weight;
    
        public Person() {
    
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public double getHeight() {
            return height;
        }
    
        public void setHeight(double height) {
            this.height = height;
        }
    
        public double getWeight() {
            return weight;
        }
    
        public void setWeight(double weight) {
            this.weight = weight;
        }
    
        private ArrayList<String> hobbies = new ArrayList<String>();
    
        public ArrayList<String> getHobbies() {
            return hobbies;
        }
    
        public void setHobbies(ArrayList<String> hobbies) {
            this.hobbies = hobbies;
        }
    
        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    ", height=" + height +
                    ", weight=" + weight +
                    '}';
        }
    
        @Override
        public Object clone() {
            Person person = null;
            try {
                person = (Person) super.clone();
                person.name = this.name;
                person.weight = this.weight;
                person.height = this.height;
                person.age = this.age;
    
                person.hobbies = (ArrayList<String>) this.hobbies.clone();
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }
            return person;
        }
    }

### 已有示例

Android中的Bundle类，该类实现了Cloneable接口

    public Object clone() {
    	return new Bundle(this);
    } 
    public Bundle(Bundle b) {
    	super(b);
    
    	mHasFds = b.mHasFds;
    	mFdsKnown = b.mFdsKnown;
    }

Intent类，该类也实现了Cloneable接口

    @Override
    public Object clone() {
    	return new Intent(this);
    }
    public Intent(Intent o) {
    	this.mAction = o.mAction;
    	this.mData = o.mData;
    	this.mType = o.mType;
    	......
    }

使用的时候可以直接拷贝现有的Intent，再修改不同的地方，便可以直接使用。

    Uri uri = Uri.parse("smsto:10086");    
    Intent shareIntent = new Intent(Intent.ACTION_SENDTO, uri);    
    shareIntent.putExtra("sms_body", "hello");    
    
    Intent intent = (Intent)shareIntent.clone() ;
    startActivity(intent);

开源库OkHttp中，也应用了原型模式。它就在OkHttpClient这个类中，它实现了Cloneable接口

    @Override 
    public OkHttpClient clone() {
    	return new OkHttpClient(this);
    }
    private OkHttpClient(OkHttpClient okHttpClient) {
    	this.routeDatabase = okHttpClient.routeDatabase;
    	this.dispatcher = okHttpClient.dispatcher;
    	this.proxy = okHttpClient.proxy;
    	......
    }

## 5. 建造者模式（Builder Pattern）-- Android常用模式

### 简介

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

### 核心Code 

-----------优化前

- 参数的构造函数的最后面的两个参数含义不清，可读性不怎么好，可能需要点开查看源码
- 当有很多参数时，编写这个构造函数就会显得异常麻烦
   
    Person p1=new Person();
    Person p2=new Person("张三");
    Person p3=new Person("李四",18);
    Person p4=new Person("王五",21,180);
    Person p5=new Person("赵六",17,170,65.4);
    
    public class Person {
        private String name;
        private int age;
        private double height;
        private double weight;
    
        public Person() {
        }
    
        public Person(String name) {
            this.name = name;
        }
    
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        public Person(String name, int age, double height) {
            this.name = name;
            this.age = age;
            this.height = height;
        }
    
        public Person(String name, int age, double height, double weight) {
            this.name = name;
            this.age = age;
            this.height = height;
            this.weight = weight;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public double getHeight() {
            return height;
        }
    
        public void setHeight(double height) {
            this.height = height;
        }
    
        public double getWeight() {
            return weight;
        }
    
        public void setWeight(double weight) {
            this.weight = weight;
        }
    }

-----------优化后

如果换一个角度，试试Builder模式，可读性可是嗖嗖的~

我们给Person增加一个静态内部类Builder类，并修改Person类的构造函数。

创建过程一下子就变得非常清晰了。对应的值是什么属性一目了然，可读性大大增强。

    Person.Builder builder=new Person.Builder();
    Person person=builder
		.name("张三")
		.age(18)
		.height(178.5)
		.weight(67.4)
		.build();

    public class PersonNew {
        private String name;
        private int age;
        private double height;
        private double weight;
    
        private PersonNew(Builder builder) {
            this.name = builder.name;
            this.age = builder.age;
            this.height = builder.height;
            this.weight = builder.weight;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public double getHeight() {
            return height;
        }
    
        public void setHeight(double height) {
            this.height = height;
        }
    
        public double getWeight() {
            return weight;
        }
    
        public void setWeight(double weight) {
            this.weight = weight;
        }
    
        public static class Builder {
            private String name;
            private int age;
            private double height;
            private double weight;
    
            public Builder name(String name) {
                this.name = name;
                return this;
            }
    
            public Builder age(int age) {
                this.age = age;
                return this;
            }
    
            public Builder height(double height) {
                this.height = height;
                return this;
            }
    
            public Builder weight(double weight) {
                this.weight = weight;
                return this;
            }
    
            public PersonNew build() {
                return new PersonNew(this);
            }
        }
    }

### 已有示例

在Android中， Builder模式也是被大量的运用。

比如常见的对话框的创建AlertDialog.Builder，ImageLoader的初始配置。

    AlertDialog.Builder builder=new AlertDialog.Builder(this);
    AlertDialog dialog=builder.setTitle("标题")
    		.setIcon(android.R.drawable.ic_dialog_alert)
    		.setView(R.layout.myview)
    		.setPositiveButton(R.string.positive, new DialogInterface.OnClickListener() {
    			@Override
    			public void onClick(DialogInterface dialog, int which) {
    
    			}
    		})
    		.setNegativeButton(R.string.negative, new DialogInterface.OnClickListener() {
    			@Override
    			public void onClick(DialogInterface dialog, int which) {
    
    			}
    		})
    		.create();
    dialog.show();

java中有两个常见的类也是Builder模式：StringBuilder和StringBuffer

还有比较著名框架中：Gson中的GsonBuilder

    GsonBuilder builder=new GsonBuilder();
    Gson gson=builder.setPrettyPrinting()
    		.disableHtmlEscaping()
    		.generateNonExecutableJson()
    		.serializeNulls()
    		.create();
    		
EventBus中也有一个Builder

    EventBus(EventBusBuilder builder) {...}

OkHttp中的

    Request.Builder builder=new Request.Builder();
    Request request=builder.addHeader("","")
    	.url("")
    	.post(body)
    	.build();
    	
    private Response(Builder builder) {...}
    	
各大框架中大量的运用了Builder模式。

小结：

- 定义一个静态内部类Builder，内部的成员变量和外部类一样
- Builder类通过一系列的方法用于成员变量的赋值，并返回当前对象本身（this）
- Builder类提供一个build方法或者create方法用于创建对应的外部类，该方法内部调用了外部类的一个私有构造函数，该构造函数的参数就是内部类Builder
- 外部类提供一个私有构造函数供内部类调用，在该构造函数中完成成员变量的赋值，取值为Builder对象中对应的值

# 设计模式专题

- [设计模式专题一：创建型模式5种](http://vivianking6855.github.io/2017/07/03/Android-Design-Pattern-1/) 
- [设计模式专题二：结构型模式7种](http://vivianking6855.github.io/2017/07/04/Android-Design-Pattern-2/) 
- [设计模式专题三：行为型模式11种](http://vivianking6855.github.io/2017/07/05/Android-Design-Pattern-3/) 

Github Code: [https://github.com/vivianking6855/android-advanced/tree/master/DesignPattern](https://github.com/vivianking6855/android-advanced/tree/master/DesignPattern)

# Reference

[Android开发中常见的设计模式](http://www.cnblogs.com/android-blogs/p/5530239.html)

[Android设计模式之23种设计模式一览](http://blog.csdn.net/happy_horse/article/details/50908439)

[《Android深入透析》之常用设计模式经验谈](https://my.oschina.net/u/2249934/blog/343441)

《android之大话设计模式》

[设计模式中英文对照](https://wenku.baidu.com/view/a216bfa651e79b896802269a.html)

[Android 设计模式](http://blog.csdn.net/banketree/article/details/24985607)