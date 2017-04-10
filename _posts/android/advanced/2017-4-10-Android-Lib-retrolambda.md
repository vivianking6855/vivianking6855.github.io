---
layout: post
title: 开源库 - Lambda插件：gradle-retrolambda
date: 2017-4-10
excerpt: "开源库 - Lambda插件：gradle-retrolambda"
categories: Android
tags: [Android 进阶]
comments: true
---

# 前言

[gradle-retrolambda](https://github.com/evant/gradle-retrolambda)

支持java1.6，java1.7和安卓的Lamdba表达式的gradle插件 

# AS 配置

虽然retrolambda github项目上说在需要从mavenCentral下载，不过jcenter()也已经有了。

## 1. 添加 dependencie： project build.gradle

    buildscript {
        repositories {
            jcenter()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:2.3.1'
            classpath 'me.tatarka:gradle-retrolambda:3.6.0'  //此处是新增
    
            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
        }
    }

## 2. 添加plugin

两种写法均可：

写法一：project build.gradle中添加

    allprojects {  
        repositories {  
            jcenter()  
            apply plugin: 'me.tatarka.retrolambda' //此处是新增  
        }  
    } 

写法二： Module:app build.gradle 中添加

    apply plugin: 'com.android.application'
    apply plugin: 'me.tatarka.retrolambda' //此处是新增  

## 3. 添加配置 Module:app build.gradle 中添加

    android {
        compileSdkVersion 25
        buildToolsVersion "25.0.0"
        compileOptions {  //此处是新增  
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }

## 4. 使用

配置好后，先sync下Project. 现在可以开始使用啦：

    new Button(this).setOnClickListener(v -> {onBackPressed();});

## 5. 注意事项

需要配置jdk 8

不过AndroidStudio 2.3.1已经很人性化的帮我们默认配置的是jdk 8了

确认方法： 

![](http://i.imgur.com/8XYt1gO.jpg)

进入你的jre下的bin文件夹，cmd命令行run： java -version . 检查下是不是1.8

D:\Program Files\Android\Android Studio\jre\bin>java -version

openjdk version "1.8.0_112-release"

OpenJDK Runtime Environment (build 1.8.0_112-release-b06)

OpenJDK 64-Bit Server VM (build 25.112-b06, mixed mode)

# Reference

> [retrolambda安装](http://blog.csdn.net/qq_26819733/article/details/52225653)
