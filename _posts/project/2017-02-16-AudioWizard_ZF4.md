---
layout: post
title: 2016-10-28 AudioWizard
date: 2016-10-28
excerpt: "AudioWizard项目总结"
project: true
tags: [Projects]
comments: true
---


# 目录
- [项目内容](#项目内容)  
- [Members](#Members) 
- [平台支持](#平台支持)
- [开发方式和架构内容](#开发方式和架构内容)
- [遇到的问题](#遇到的问题)
- [项目优化](#项目优化)
- [其他](#其他)
    - [项目档案](#项目档案)
- [总结](#总结)

---
<h1 id="项目内容"> 项目内容 </h1>

AudioWizard 音频向导。 

- 三种scenario可支持多种GEQ音效：Norma,Pop,Rock等
    - BT
    - HP
    - Speaker
- 耳机450+可选
- 听力补偿
- DTS音场效果

---
<h1 id="Members"> Members </h1>
- venjee
- Raygee

---
<h1 id="平台支持"> 平台支持 </h1>

Android版本已发布，设备出厂预装

---
<h1 id="开发方式和架构内容"> 开发方式和架构内容 </h1>

1. Android开发
 - 搭配BSP接口实现多种音效和配置
 - 搭配DTS厂商库实现底层音效
2. 使用Git作为版本管理
3. 使用Android Studio开发客户端
4. 随机器预装方式升级方式
5. 使用Android官网，github，Stackoverflow和codekk作为主要的知识查找
6. 使用Lint, FindBugs, monkey test做app代码和quanlity检查


---
<h1 id="遇到的问题"> 遇到的问题 </h1>

- DTS database 有图版本size过大 > 8M

    解决方案：
    
    - 请DTS提供无icon版本。
    - search “DISTINCT Type,SubType” from database,  22 items list as below. app自行预载配件对应icon。

---
<h1 id="项目优化"> 项目优化 </h1>


---
<h1 id="其他"> 其他 </h1>

<h2 id="项目档案"> 项目档案 </h2>

- 项目进度：[F:\Manager\AudioWizard](F:\Manager\AudioWizard)

- 项目档案：[F:\Manager\AudioWizard](F:\Manager\AudioWizard)


---
<h1 id="总结"> 总结 </h1>

- AudioWizard 分View 和 Service两支app
  
    Service负责音效处理。View主要负责页面显示
    


> [Android播放audio音频踩坑实践](http://www.jianshu.com/p/fee65523a632)

> [google mediaplayer](https://developer.android.com/guide/topics/media/mediaplayer.html#mediaplayer)