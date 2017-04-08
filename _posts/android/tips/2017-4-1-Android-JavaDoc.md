---
layout: post
title: Tool - JavaDoc
date: 2017-4-1
excerpt: "Tint"
categories: Android
tags: [Android 微技巧]
comments: true
---

#  Android studio JavaDoc的使用

- 安装

# 安装插件Javadoc

- File → Settings → Plugins → Browse repositories 搜索Javadoc
- 点击install
- 重启androidstudio

# 使用

alt+insert 或者是选择文件右键 JavaDocs

# 生成JavaDoc文档

对于androidlibrary来说，如果生成JavaDoc文档建议删掉build文件以及测试文件。

打开Tools--->Generate JavaDoc

因为很多文件会是UTF-8的编码格式，所以建议加上

-encoding utf-8 -charset utf-8 (Other command line arguments)
 

<br/>
<br/>


> [ Android studio JavaDoc的使用](http://blog.csdn.net/dreamlivemeng/article/details/51499675)
