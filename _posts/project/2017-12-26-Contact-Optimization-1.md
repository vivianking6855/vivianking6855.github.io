---
layout: post
title: Contact 优化 - 开篇
date: 2017-12-26
excerpt: "Contact 优化 - 开篇"
categories: Projects
tags: [Projects]
comments: true
---


* content
{:toc}




# 简介

app优化主要包含size优化，性能优化，重构等。

其中性能优化又是特别重要的一环。性能主要关注：

- 内存
- CPU
- 耗电
- 卡顿
- 渲染
- 进程存活率等

性能优化需要注意：

1. 不要过早的做性能优化，app先求能用再求好用。在需求都还没完成的时候，花大量时间在优化上是本末倒置的
2. 优化要用实际数据说话，建议借助测试工具进行检测。检测工具参看[这里](http://vivianking6855.github.io/2017/12/26/Android-optimization-Tool/)

总之，要合理优化，数据量化。

# Contact优化

我们针对联系人做的优化包括：

1. size优化： 移除unused resources，降低app的size
2. 布局，绘制，响应速度等性能优化

    - 联络人详情优化: 主要是布局优化，绘制优化，响应速度优化，内存优化
    - 主页面优化: 主要是布局优化，绘制优化，响应速度优化，内存优化
    - call log滑动: 主要是布局优化，绘制优化，响应速度优化，内存优化
    - smart search: 内存和响应速度优化

3. 响应时间（Response Time） 

    相应标准步骤和标准如下：
    
    测试工具：高速摄像机
    
    测试条件：
    
    - DUT存在500筆連絡人, 每個連絡人皆有頭像 (使用test file: 500(photo).vcf)
    
    测试项目：
    
    - Contacts Preview Window launch  time. 联系人预览时间测试。 要求0.6s内
        - 1. Kill Contacts process (無需重開機)
        - 2. 點擊Contacts, 當手指離開螢幕時即開始計時, 當Contacts的"底圖"完成顯示即停止計時（不包含底图的文字和icon）
    - Launch time of Contacts (1st) 联系人页面加载时间。 要求2s内
        - 1. Kill Phone & Contacts process (無需重開機)
        - 2. 開啟Contacts, 當手指離開螢幕時即開始計時
        - 3. 當連絡人畫面完全顯示後,停止計時
    - Frame rate of scrolling contacts. 联系人滑动卡顿测试。 要求50帧内
        - 1. 開啟連絡人app
        - 2. 滑動聯絡人以載入聯絡人頭像
        - 3. 使用手指向上滑動, 計算畫面捲動的frame rate
    - Frame rate of scrolling call log. 联系人滑动卡顿测试。 要求50帧内
        - 1. 開啟Phone app > 將撥號鍵收起來 (秀出整頁的call log)
        - 2. 滑動call log以載入聯絡人頭像
        - 3. 使用手指向上滑動, 計算畫面捲動的frame rate
    - Response time of backing to Home。 联系人退出时间测试。要求0.8s内
        - 1.新增10筆聯絡人資料(均需有頭像、姓名、電話)
        - 2.切換到Favorite tab，選擇step1建立的10筆資料設為Favorite
        - 3.在Favorite頁面點擊Home鍵, 當手指離開Home鍵即開始計時, 至回到Home畫面

3. [重构](http://vivianking6855.github.io/2017/03/30/Android-Design-Refactoring/)，在现有的框架上适当重构。

4. 其他
    - zipAlignEnabled: 对应用程序中的资源作对齐操作 
    
        使得在运行时Android与应用程序间的交互更加有效率。这种方式能够让应用程序和整个系统运行得更快
    
            buildTypes {
                release {
                    shrinkResources true
                    minifyEnabled true
                    zipAlignEnabled true
                    proguardFiles 'proguard.flags'
                }
            }

# Reference

[性能优化 - 目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)