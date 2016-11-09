---
layout: post
title: 2016-09-14 Android 之 微技巧 （一）
date: 2016-09-14
excerpt: "Snackbar, CoordinatorLayout, Annotations,热更新"
tags: [Android]
comments: true
---

## 1. Dialog, Toast, Snackbar

[Dialog, Toast, Snackbar](http://blog.csdn.net/guolin_blog/article/details/51336415)

## 2. Android状态栏微技巧，带你真正理解沉浸式模式

[沉浸式模式](http://blog.csdn.net/sinyu890807/article/details/51763825)

## 3. Android drawable

[你所不知道的drawable的那些细节](http://blog.csdn.net/sinyu890807/article/details/50727753)

## 4. CoordinatorLayout ：super-powered FrameLayout

[CoordinatorLayout](http://blog.csdn.net/xyz_lmn/article/details/48055919)


## 5. Annotations

[Java注解Annotation基础](http://www.open-open.com/lib/view/open1423558996951.html)：注解是一种元数据，起到了”描述，配置“的作用。

### 注解框架 和 编译时注解的框架

#### AndroidAnnotations - 注解框架

[AndroidAnnotations](http://my.oschina.net/jack1900/blog/296953)是一个开源框架，旨在加快Android开发的效率。

通过使用它开放出来的注解api，你几乎可以使用在任何地方， 大大的减少了无关痛痒的代码量，让开发者能够抽身其外，有足够的时间精力关注在真正的业务逻辑上面。

而且通过简洁你的代码，也提高了代码的稳定性和后期的维护成本。

目前主流的注解框架有：

- xUtils
- ButterKnife
- Dragger
- Roboguice

它们的实现原理都是一致的，都是通过反射机制实现的。

#### 编译时注解的框架

在Android应用开发中，我们常常为了提升开发效率会选择使用一些基于注解的框架，但是由于反射造成一定运行效率的损耗，

所以我们会更青睐于编译时注解的框架。常用的编译时注解的框架有：

- butterknife免去我们编写View的初始化以及事件的注入的代码。
- EventBus3方便我们实现组建间通讯。
- fragmentargs轻松的为fragment添加参数信息，并提供创建方法。
- ParcelableGenerator可实现自动将任意对象转换为Parcelable类型，方便对象传输。

类似的库还有非常多，大多这些的库都是为了自动帮我们完成日常编码中需要重复编写的部分。

当然并不是说上述框架就一定没有使用反射了，其实上述其中部分框架内部还是有部分实现是依赖于反射的，但是很少而且一般都做了缓存的处理。所以相对来说，效率影响很小。

## 6. android热更新技术研究 
[android热更新技术研究](http://blog.csdn.net/qq_25943493/article/details/51463884)  


<br>
<br>

> [Android 如何编写基于编译时注解的项目](http://blog.csdn.net/lmj623565791/article/details/51931859)



