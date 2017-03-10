---
layout: post
title: AudioWizard_ZF4
date: 2017-2-16
excerpt: "AudioWizard_ZF4项目总结"
categories: Projects
tags: [Projects]
comments: true
---

* content
{:toc}



# 一、 项目内容

AudioWizard 音频向导。 

- 三种scenario可支持多种GEQ音效：Norma,Pop,Rock等
    - BT
    - HP
    - Speaker
- 耳机450+可选
- 听力补偿
- DTS音场效果

Members

- venjee
- Raygee

# 二、 平台支持

Android版本已发布，设备出厂预装

# 三、 开发方式和架构内容

1. Android开发
 - 搭配BSP接口实现多种音效和配置
 - 搭配DTS厂商库实现底层音效
2. 使用Git作为版本管理
3. 使用Android Studio开发客户端
4. 随机器预装方式升级方式
5. 使用Android官网，github，Stackoverflow和codekk作为主要的知识查找
6. 使用Lint, FindBugs, monkey test做app代码和quanlity检查

# 四、 数据缓存机制

# 五、 安全

# 六、 遇到的问题

- DTS database 有图版本size过大 > 8M

    解决方案：
    
    - 请DTS提供无icon版本。
    - search “DISTINCT Type,SubType” from database,  22 items list as below. app自行预载配件对应icon。

# 七、 项目优化

# 八、 其他

## 项目档案

- 项目进度：[F:\Manager\AudioWizard](F:\Manager\AudioWizard)

- 项目档案：[F:\Manager\AudioWizard](F:\Manager\AudioWizard)

# 九、 总结

- AudioWizard 分View 和 Service两支app
  
    Service负责音效处理。View主要负责页面显示
    
# 十、 Reference

> [Android播放audio音频踩坑实践](http://www.jianshu.com/p/fee65523a632)

> [google mediaplayer](https://developer.android.com/guide/topics/media/mediaplayer.html#mediaplayer)