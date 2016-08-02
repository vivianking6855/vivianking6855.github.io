---
layout: post
title: ReactNative
date: 2016-07-15
excerpt: "如何解决Android应用方法数不能超过65K的问题"
tags: [Android]
comments: true
---

## Introduction

介绍如何解决Android 应用方法数不能超过65K的问题

google官网的做法可参看 [Configure Apps with Over 64K Methods](https://developer.android.com/studio/build/multidex.html)

从5.0开始支援Multidex

Configuring Your App for Multidex with Gradle，need to perform the following steps:

- Change your Gradle build configuration to enable multidex
- Modify your manifest to reference the MultiDexApplication class



关键Code:
    
    1. modify app\build.gradle
    android {
        compileSdkVersion 21
        buildToolsVersion "21.1.0"
    
        defaultConfig {
            ...
            minSdkVersion 14
            targetSdkVersion 21
            ...
    
            // Enabling multidex support.
            multiDexEnabled true
        }
        ...
    }
    
    dependencies {
      compile 'com.android.support:multidex:1.0.0'
    }


    2. AndroidManifest.xml 添加新的Application入口： MultiDexApplication
     <application
          android:allowBackup="true"
          android:label="@string/app_name"
          android:icon="@mipmap/ic_launcher"
          android:name="android.support.multidex.MultiDexApplication"



> [Build Apps with more than 65,000 methods，](http://developer.android.com/tools/building/multidex.html)

