---
layout: post
title: 2015-10-01 Flashlight
date: 2015-10-01
excerpt: "Flashlight项目总结"
project: true
tags: [Projects]
comments: true
---


# 目录
- [项目内容](#项目内容)  
- [平台支持](#平台支持)
- [开发方式和架构内容](#开发方式和架构内容)
- [其他](#其他)
    - [项目档案](#项目档案)
- [总结](#总结)

---
<h1 id="项目内容"> 项目内容 </h1>
Android 手电筒工具。 Unbundle to ALL。

- Flashlight app：多频率闪烁，colorful，定时关闭等
- widget
- cover flashlight.

---
<h1 id="平台支持"> 平台支持 </h1>
Android版本已发布[google paly](https://play.google.com/store/apps/details?id=com.asus.flashlight) 

- 安装量27,262,899
- store评分4.6
- 排名Top 10

---
<h1 id="开发方式和架构内容"> 开发方式和架构内容 </h1>
1. Android开发
 - 搭配framework节点接口开关Flashlight
2. 使用Git作为版本管理
3. 使用Android Studio开发客户端
4. 使用Play Store和升级服务器作为升级方式
5. 使用Android官网，github，Stackoverflow和codekk作为主要的知识查找
6. 使用Lint, FindBugs, monkey test做app代码和quanlity检查


---
<h1 id="其他"> 其他 </h1>

<h2 id="项目档案"> 项目档案 </h2>

- [项目进度](N:\Project\Manager\FlashLight) 

- [项目档案](N:\Project\Manager\FlashLight)


---
<h1 id="总结"> 总结 </h1>
- 主要是项目维护和新功能开发
- Android默认Camera API速度在频率闪烁时，效率太低。优化为framework层直接提供节点接口来开关灯

