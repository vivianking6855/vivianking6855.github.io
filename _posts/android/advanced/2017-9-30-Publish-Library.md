---
layout: post
title: 如何发布库到Jcenter
date: 2017-9-29
excerpt: "如何发布库到Jcenter"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

当开发了很好的控件和库，如何更方便的跟大家分享呢？

这就是今天的我们的目标：如何用bintray-release工具发布到JCenter（AS现在默认repository是JCenter，不再是Maven了）

# step 1 注册帐号

注册帐号，[点击这里](https://bintray.com/signup/oss)

也可以直接使用github账号注册登录。注册完成之后，我们把API Key存下来，后面发布会用到： 

![](https://i.imgur.com/OINhdXi.jpg)

# step 2 创建organization,repository，项目

新版bintray增加了organization这项，网上很多之前的脚步都不能使用了。

我们在这里创建的organization和项目，用用到step 3中配置文件中。repository必须是maven（发布工具限制，我暂时没找到这个工具可以制定repo的接口）

创建完成后是这样的，红色划掉的是organization的名字

![](https://i.imgur.com/h660dUb.jpg)

![](https://i.imgur.com/1xxzYPT.jpg)

VCS我用的github上code地址

# step 3 配置工具[bintray-release](https://github.com/novoda/bintray-release)

我们使用工具bintray-release来发布

1. 在Project的build.gradle中添加一行 com.novoda:bintray-release:0.5.0

        buildscript {
            repositories {
                jcenter()
            }
            dependencies {
                classpath 'com.android.tools.build:gradle:2.3.0'
                classpath 'com.novoda:bintray-release:0.5.0' // publish tool
        
                // NOTE: Do not place your application dependencies here; they belong
                // in the individual module build.gradle files
            }
        }
        
2. 在Module(要发布的lib) build.gradle中加入

        apply plugin: 'com.novoda.bintray-release' // publish tool
        
        // publish
        publish {
            userOrg = 'xxx'//step2 创建的organization名字 
            groupId = 'com.open'// group id, jcenter上的路径,即lib使用的时候的groupid（step 5中有说明）
            artifactId = 'utilslib'//step2 创建的项目名称
            publishVersion = '1.0.0'//发布版本号
            desc = 'this is for test'//描述，可选
            website = 'https://github.com/kuyue'//网站，可选
        }

# step 4 发布

打开Android Studio的Terminal面板进行，执行下面的命令： 

windows系统： 

    gradlew clean build bintrayUpload -PbintrayUser=xxx -PbintrayKey=xxxxxxxxxxxxxxxxxxxxxx -PdryRun=false

Mac系统：

    ./gradlew clean build bintrayUpload -PbintrayUser=xxx -PbintrayKey=xxxxxxxxxxxxxxxxxxxxxx -PdryRun=false

参数说明：

- user：Jcenter用户名
- key：step 1中拿到的API key
- PdryRun：配置参数。 true 代表会运行所有的环节，但是不会上传。（在上传之前可以先设置为true，看是否可以build成功）

![](https://i.imgur.com/oGIveBx.jpg)

上传成功会在maven repository看到

![](https://i.imgur.com/HHdYI9L.jpg)

最后发布到Jcenter，点击utilslib进入详情页面，右侧Add to JCenter

![](https://i.imgur.com/Zk9KIUm.jpg)

等一天左右就可以看到，发布成功：

![](https://i.imgur.com/NfzQBvj.jpg)

# step 5 在项目中使用发布的库






# 遇到的问题

1. Could not find tools.jar. 

    ![](https://i.imgur.com/mb4xe5f.jpg)

    解决方案：在gradle.properties 中配置jdk路径org.gradle.java.home=C:\\Program Files\\Java\\jdk1.8.0_91

2. 中文乱码

    ![](https://i.imgur.com/nvGNPEx.jpg)
    
    解决方案：在Project的build.gradle中配置comment编码
    
        allprojects {
            repositories {
                jcenter()
            }
        
            tasks.withType(Javadoc) {
                options{
                    encoding "UTF-8"
                    charSet 'UTF-8'
                    links "http://docs.oracle.com/javase/7/docs/api"
                }
            }
        }
        
3.  Could not create package 'open.com/maven/utilslib'

    ![](https://i.imgur.com/3ATtWAF.jpg)

    解决方案：手动创建，红色部分不能修改
    
    ![](https://i.imgur.com/BbmEl4e.jpg)
    
    ![](https://i.imgur.com/hEV2hvB.jpg)
    

# 参考资料

- [扫盲Android Studio 仓库jCenter并发布自己的开源库](http://blog.csdn.net/u013231041/article/details/70174354)
- [使用gradle发布Android studio lib库到jCenter代码库](http://blog.csdn.net/chentong2419/article/details/47981713) 
- [Android Library项目发布到JCenter最简单的配置方法](http://www.cnblogs.com/shiwei-bai/archive/2015/11/24/4991636.html)
- [新版Bintray,如何使用Gradle发布项目到Jcenter仓库](http://www.jianshu.com/p/e2cc4f66b1e7)

http://blog.csdn.net/chentong2419/article/details/47981713


