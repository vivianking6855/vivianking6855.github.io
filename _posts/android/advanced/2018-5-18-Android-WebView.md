---
layout: post
title: Android Webview小结
date: 2016-11-01
excerpt: "Android Webview小结"
categories: Android
tags: [Android 进阶]
comments: true
---

* content
{:toc}



# 简介

Webview是Hybrid开发中非常重要的知识点，下面对用到的Webview知识做一个小结

   
# 一、 JS和Java的安全交互   

JsBridge, JS和Native交互的桥梁。在Android开发中，能实现Js调用Java，有4种方法：

1. JavascriptInterface
2. WebViewClient.shouldOverrideUrlLoading()
3. WebChromeClient.onConsoleMessage()
4. WebChromeClient.onJsPrompt()

[JsBridge实现及原理](https://blog.csdn.net/u013262934/article/details/70210147)

[Android混合开发之WebViewJavascriptBridge实现JS与java安全交互](https://www.cnblogs.com/whoislcj/p/6104015.html)

[WebView深度学习（一）之WebView的基本使用以及Android和js的交互](https://www.jianshu.com/p/b9164500d3fb)

滑动冲突

1. H5 banner和 Native ViewPager滑动冲突

    解决方案：上面H5 banner的上面区域禁止ViewPager滑动

2. 推荐页面H5 “click to top” button 点击事件触发背景点击

    推荐页面的to top button点击后，事件会穿透到下面的listview. 导致点击to top后会进入帖子正文。
    
    解决方案：在image div后面叠加一个div，并在hide的时候延迟一下。css的touch会在300ms后出发click，因此这里400ms后再把遮挡div hide掉。
    
        .html----------------
        <div id="hide" class="float_icon">
        </div>
        <div id = "totop" class="float_icon">
            <img src="images/btn_top.png" width="100%" height="100%">
        </div>
    
        .js-----------------
        
        $(document).scroll(function(){
            if($(document).scrollTop()>0){
                hide.show();
                floatIcon.show();
            }else{
                setTimeout("hide.hide()",400);
                floatIcon.hide();
            }
        });


# 缓存 

1. webview Cache

       App端方案：加入webview cache方案。 实际测试效果：（通过抓包和图片文件分析）
        
       1）	图片不会重新load
        
       2）	网页相关的html, css, js等会重新下载
        
       不过这些文件加起来基本< 10k，对加载效率影响已经非常小。
       设定cache方案后，为何这些文件会重新下载还待进一步研究。初步猜测可能跟服务器端配置有关。

	   webview缓存跟服务器配置有关，一般webview设定后就可以支援断网后显示缓存数据
	       
		       private void addWebViewCacheEnable(WebView webView){
		            WebSettings webSettings = webView.getSettings();
		            webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);//LOAD_DEFAULT
		            webSettings.setDomStorageEnabled(true);
		            webSettings.setDatabaseEnabled(true);
		            webSettings.setAppCachePath(getContext().getCacheDir().getAbsolutePath());
		            webSettings.setAppCacheEnabled(true);
		       }


	[安卓WebView相关设置](http://frank-zhu.github.io/android/2015/08/19/android-html5-web-view/)

	[手把手教你构建 Android WebView 的缓存机制 & 资源预加载方案](https://blog.csdn.net/carson_ho/article/details/71402764)

2. session过期问题
   
   ZenTalk遇到登录一段时间后无法发帖问题,经查证是session的问题。首先跟服务器端确认如下：
   
   - formsession相当于页面session， 只保存在 client 端。每次登陆都会生成一个formhash 值,只要webview 不关闭这个值将一直有效。
   - 如果apk 关闭，再重新打开，理论上会引起 formhash 值失效。需要调用zentalkApp_v1.2.8_SPEC.doc 文档中的init Api 重新获取 fromhash值。
   - 如果在asusaccount 页面做logout 操作，则formhash 会失效。

   解决方案： app每次启动后重新获取formsession
  

# 性能和优化

- webview采自用不动加载图片，等页面finish后再发起图片加载方案。提升webview的UX
- webview加载优化之缩略图

        建议服务器支援多档图片，例如
        
        1)	Thumbnail：小图，48*48, 92*92?  用于组文章，帖子正文等页面，加速帖子列表渲染和加载
        
        2)	preview大图：512*512, 1024*768?     用于user点击后查看大图，加速大图页面的渲染和加载
        
        3)	原图：已经支援

- webview 内存泄漏

	[WebView深度学习（三）之WebView的内存泄漏、漏洞以及缓存机制原理和解决方案](https://www.jianshu.com/p/44b977907e51)

	webView的彻底销毁

		if (mWebView != null) {
	        mWebView.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
	        mWebView.clearHistory();
	
	        ((ViewGroup) mWebView.getParent()).removeView(mWebView);
	        mWebView.destroy();
	        mWebView = null;
	    }


[WebView内存泄漏，如何让WebView清除更彻底](https://blog.csdn.net/qq_16318981/article/details/45362399)

[Android 彻底关闭WebView，防止WebView造成OOM](https://blog.csdn.net/yaphetzhao/article/details/48521581)

[Android WebView：性能优化不得不说的事](https://www.aliyun.com/jiaocheng/51101.html)

[WebView深度学习（二）之全面总结WebView遇到的坑及优化](https://www.jianshu.com/p/2b2e5d417e10)

[WebView深度学习（三）之WebView的内存泄漏、漏洞以及缓存机制原理和解决方案](https://www.jianshu.com/p/44b977907e51)

# 三方库

[腾讯TBS X5 WebView的简单使用](https://blog.csdn.net/qq_27634797/article/details/76622590)

[safe-java-js-webview-bridge](https://github.com/pedant/safe-java-js-webview-bridge): 通过onJsPrompt()实现JsBridge

[UrlRouter](https://github.com/lzyzsd/JsBridge):重写shouldOverrideUrlLoading（）方法
	
# Reference

[《移动端本地 H5 秒开方案探索与实现》](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653579803&idx=1&sn=0a1f418f628a53e9262b97879f47593b&chksm=84b3b81cb3c4310a175b95264bf74a5be8abf39751ed0b21a353e68e6cf1846db11995bc2fec&mpshare=1&scene=1&srcid=0524GVlrCE03TcvbOGYNMFHw#rd)

[WebView性能、体验分析与优化](https://tech.meituan.com/WebViewPerf.html)

[Android WebView开发问题及优化汇总 - 阿里云](https://www.aliyun.com/jiaocheng/655803.html)

[Android Webview太烂？试试Webview独立进程吧](https://www.jianshu.com/p/8ed995016fde)

[Android WebView 全面干货指南](https://www.jianshu.com/p/fd61e8f4049e)

[Android WebView开发问题及优化汇总](http://blog.csdn.net/xyz_lmn/article/details/3947339473701701)

[Android WebView 缓存处理](http://www.open-open.com/lib/view/open1392188052301.html)

[WebView控件之WebSettings详解](https://www.jianshu.com/p/0d7d429bd216)