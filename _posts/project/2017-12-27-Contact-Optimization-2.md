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

1. 选择工具：“Remove Unused Resources”

    ![选择工具](https://github.com/vivianking6855/vivianking6855.github.io/blob/master/datum/images/AS/refactor_1.jpg?raw=true)

2. Refactor前建议先Preview查看一下待移除的资源，做一次确认

    ![](https://github.com/vivianking6855/vivianking6855.github.io/blob/master/datum/images/AS/refactor_2.jpg?raw=true)

    Contact执行操作后共移除1971项

3. 编译apk，验证app

    发现app触宝相关功能会crash. 
   
    因为触宝的.jar会通过反射的方式使用资源(aar可以避免这类问题，因为aar可以带资源)
   
4. 修复问题，再次验证app

    恢复所有触宝用到的资源，验证app可正常运行  

5. 其他
    
    除了移除unused resources外，还有一些可能影响app size：
    
    - icon建议使用svg格式适配所有型号的专案
    - 无需透明度图片建议使用jpg而非png图片
    - 使用针对智能手机优化过的三方库
    - 启用ProGuard 压缩
   
            buildTypes {
                release {
                    minifyEnabled true //启用代码压缩
                    shrinkResources true //启用资源压缩
                    ......
                }
            }


# 优化效果对比

共节省size： 

PS：size以Jenkins build出的apk为准

# 注意事项

需要特别注意以下两种情形，AS工具不会纳入检查，需要手动确认

1. jar包中通过反射使用的resources
2. 外部app通过反射使用的resources

比如Contact中触宝jar包会使用到的resources，在使用工具后会被删除掉，需要手动恢复回来。

