---
layout: post
title: Android Material Design Theme
date: 2018-2-28
excerpt: "Android Material Design Theme"
categories: Android
tags: [Android 基础]
comments: true
---



# 使用材料主题

材料主题的定义为：

    @android:style/Theme.Material（深色版本）
    @android:style/Theme.Material.Light（浅色版本）
    @android:style/Theme.Material.Light.DarkActionBar

可使用的材料风格，请参阅 [R.style](https://developer.android.com/reference/android/R.style.html?hl=zh-cn) 的 API 参考

颜色定制

    <resources>
      <!-- inherit from the material theme -->
      <style name="AppTheme" parent="android:Theme.Material">
        <!-- Main theme colors -->
        <!--   your app branding color for the app bar -->
        <item name="android:colorPrimary">@color/primary</item>
        <!--   darker variant for the status bar and contextual app bars -->
        <item name="android:colorPrimaryDark">@color/primary_dark</item>
        <!--   theme UI controls like checkboxes and text fields -->
        <item name="android:colorAccent">@color/accent</item>
      </style>
    </resources>


![](https://developer.android.com/training/material/images/ThemeColors.png?hl=zh-cn)


[使用材料主题](https://developer.android.com/training/material/theme.html?hl=zh-cn) 