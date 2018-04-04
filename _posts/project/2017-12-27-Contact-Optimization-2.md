---
layout: post
title: Contact 优化 - size优化
date: 2017-12-27
excerpt: "Contact 优化 - size优化"
categories: Projects
tags: [Projects]
comments: true
---


* content
{:toc}



# 简介

size优化是移除unused resources，降低app的size。

可以节省设备空间，同时也可以提升app在store上的下载几率。

# 工具

size优化会用到Android Studio自带工具： “Remove Unused Resources”

# 优化实践

1. 使用工具：“Remove Unused Resources” 移除Unused资源

    - 1.1 选择工具
    
        ![选择工具](https://github.com/vivianking6855/vivianking6855.github.io/blob/master/datum/images/AS/refactor_1.jpg?raw=true)

    - 1.2 Preview确认
    
        Refactor前建议先Preview查看一下待移除的资源，做一次确认

        ![Preview](https://github.com/vivianking6855/vivianking6855.github.io/blob/master/datum/images/AS/refactor_2.jpg?raw=true)

        Contact执行操作后共移除1971项

    - 1.3 编译并验证

        发现app触宝相关功能会crash. 
   
        因为触宝的.jar会通过反射的方式使用资源(aar可以避免这类问题，因为aar可以带资源)
   
    - 1.4 修复问题，再次验证

        恢复所有触宝用到的资源，验证app可正常运行  

2. 移除pad对应的resources
    
    因Pad资源不再使用，因此移除Contact的Pad资源，包含value, layout, drawable下面的
    
    - sw580
    - sw680
    - sw800    

3. 其他
    
    除了移除unused resources外，针对减小app size还有下面的建议：
    
    - icon建议使用svg格式适配所有型号的专案
    - 无需透明度图片建议使用jpg而非png图片
    - 使用针对智能手机优化过的三方库
    - 启用ProGuard 压缩
   
            buildTypes {
                release {
                    minifyEnabled true //启用代码压缩,混淆
                    shrinkResources true //启用资源压缩
                    ......
                }
            }

    PS: 如果仅需要代码压缩，不做混淆，可以proguard文件中制定：-dontobfuscate

# 优化效果对比

优化前size: 29,715KB	

优化后size: 25,478KB	

共节省size： <b> 4.237M (4237KB) </b>

PS：size以Jenkins build出的apk为准 （proguard压缩过的大小）

# 注意事项

需要特别注意以下两种情形，AS工具不会纳入检查，需要手动确认

1. jar包中通过反射使用的resources
2. 外部app通过反射使用的resources

比如Contact中触宝jar包会使用到的resources，在使用工具后会被删除掉，需要手动恢复回来。

# 测试

优化完成后必不可少的步骤是测试，依据前面修改范围，有几处需要重点测试

1. Font size change
2. Display size change
3. 不同resolution 设备UI展示测试

当然还有比不可少的monkey测试

# Reference

[性能优化 - 目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)