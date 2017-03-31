---
layout: post
title: WorkClub
date: 2017-3-31
excerpt: "WorkClub项目总结"
categories: Projects
tags: [Projects]
comments: true
---

* content
{:toc}


# 一、 项目内容

[墨刀地址](https://modao.cc/app/PRizIEov59JZ6Fs6NhISvQKcW3iFH0O?inapp=1)

1. 登录
2. 显示不同板块数据
3. 聊天
4. 发帖
5. 搜索

# 二、 平台支持

iOS和Android

Android已经上架1版

# 三、 开发方式和架构内容

1. 使用Git作为版本管理
2. 使用Android Studio，VS Code，WebStorm开发客户端
3. 使用Laravel框架开发服务器
3. 使用Microsoft CodePush作为js package的升级工具
4. 使用Redux作为React的数据框架
5. 使用Stackoverflow和RN 官网，js.coach作为主要的知识查找
6. 开发了一些自己使用的Android的插件：城市，地图。因为RN效率和地图定位不准确问题
7. 主要使用Android开发，然后IOS适配
8. 使用eslint做js的静态检查

# 四、 数据缓存机制

# 五、 安全

# 六、 遇到的问题

# 七、 项目优化

# 八、 其他

## 使用到的第三方库

-  "lodash": "^4.13.1"
-  "react-immutable-render-mixin": "^0.9.7"
-  "react-native-action-button": "^1.1.5"
-  "react-native-alphabetlistview": "^0.2.0",
-  "react-native-animatable": "^0.6.1",
-  "react-native-code-push": "^1.13.5-beta",
-  "react-native-datepicker": "^1.3.0",
-  "react-native-gcm-android": "^0.2.0",
-  "react-native-image-crop-picker": "^0.4.0",
-  "react-native-image-picker": "^0.19.5",
-  "react-native-modalbox": "^1.3.4",
-  "react-native-networking": "^0.1.1",
-  "react-native-open-share": "git+https://github.com/mozillo/react-native-open-share.git",
-  "react-native-progress": "^3.0.1",
-  "react-native-rong-imlib": "git+https://github.com/reactnativecn/react-native-rong-imlib.git",
-  "react-native-scrollable-tab-view": "^0.5.2",
-  "react-native-storage": "^0.1.2",
-  "react-native-swiper": "^1.4.6",
-  "react-native-system-notification": "^0.1.11",
-  "react-native-theme": "^0.1.2",
-  "react-native-timer": "^1.1.2",
-  "react-native-vector-icons": "^2.0.3",
-  "react-redux": "^4.4.5",
-  "redux": "^3.5.2",
-  "redux-immutable": "^3.0.6",
-  "redux-logger": "^2.6.1",
-  "redux-thunk": "^2.1.0"

# 九、 总结

1. Android的坑是有不少，比如：首页白屏，list滚动效率，地图定位不准确
2. Android的库太少
3. Android的性能好像也不是很好，但是，也能凑合用
4. Android的原生控件封装的不好
5. 如果希望代码复用高，最好让iOS和Android尽量保持样式的一致


