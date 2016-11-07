---
layout: post
title: 2016.10 AudioWizard
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
- [其他](#其他)
    - [项目档案](#项目档案)
- [总结](#总结)

---
<h1 id="项目内容"> 项目内容 </h1>
AudioWizard 音频向导。 支援很多音频特效：

- Music模式
- 影院模式
- 游戏模式
- 演讲模式
- 智能模式
- 户外

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
 - 搭配各厂商库实现底层音效
2. 使用Git作为版本管理
3. 使用Android Studio开发客户端
4. 随机器预装方式升级方式
5. 使用Android官网，github，Stackoverflow和codekk作为主要的知识查找
6. 使用Lint, FindBugs, monkey test做app代码和quanlity检查


---
<h1 id="遇到的问题"> 遇到的问题 </h1>

- Visualizer issue（详见[档案Visulizer......](F:\Manager\AudioWizard\Datum)）
    - Visualizer 釋放問題，底層執行緒問題造成AW crash. google defect issue.

        google issue ID 200020, [Type-Defect.](https://code.google.com/p/android/issues/detail?id=200020&can=1&q=Visualizer&colspec=ID%20Status%20Priority%20Owner%20Summary%20Stars%20Reporter%20Opened  ) 
        
        解决方案：BSP porting 对应patch

    - Visualizer 釋放問題，底層記憶體釋放問題。 

        google defect issue. google issue ID 175143, [Type defect](https://code.google.com/p/android/issues/detail?id=175143)

        解决方案： app端取消硬體加速後，crahs有大幅降低（2/1 -> 40/1）。不過波形圖會稍有卡頓
        
- 动态支援不同设备的某些功能
  
      AudioWizard是apk release到不同专案。因各专案对耳機列表有不同的需求。因此为了有更多靈活性和更好的擴展性。
      
      采用設定"DTS tuning_configs"  +  “property” 方案，实现动态改變支援列表。無需更新AW. 只要BSP配置特定的configs文件和property。
      
      并新建confluence 对各专案动态支援耳机列表做记录和监测：AudioWizard Confluence - AMAX Team
  
- 不同专案定制产品
  
      AudioWizard是apk release到不同专案。因个别定制专案对manifest，resource有不同需求。
      
      此次是因为个别专案需要移除QuickSettings上icon。
      
      解决方案是gradle productFlavors 搭配内部jekins build系统规则，build出特殊专案对应apk，并自动release到对应专案。
      
      详见档案整理：F:\Manager\AudioWizard\Datum\QuickSettings

---
<h1 id="其他"> 其他 </h1>

<h2 id="项目档案"> 项目档案 </h2>

- 项目进度：[F:\Manager\AudioWizard](F:\Manager\AudioWizard)

- 项目档案：[F:\Manager\AudioWizard](F:\Manager\AudioWizard)


---
<h1 id="总结"> 总结 </h1>
- AudioWizard 分View 和 Service两支app
  
    Service负责音效处理。View主要负责页面显示
- SurfaceView自定义控件优化波形图显示
