---
layout: post
title: ReactNative
date: 2016-05-25
excerpt: "ReactNative Windows下npm设定小结"
tags: [react]
comments: true
---

## npm 安装
npm install --save libname (from network)
npm install --save "E:\Download\react-native-router-master" (from local)

## npm升级

npm可以很方便的升级各种插件。

例如如果想升级express框架，使用如下命令

npm update express （非全局安装）
npm update -g express （express全局安装）

也可以使用安装命令来重装，在这里是等效于update：npm install -g express

但是如果想升级npm自身，可以先cd到nodejs安装目录：（例如： D:\Program Files\nodejs）

cd "D:\Program Files\nodejs"

npm update npm

常用命令：
npm info react-native
npm update -g react-native-cli：更新cli

## react-native升级
npm info react-native：查看新版信息
react-native --version：查看本地信息
修改package.json中的依赖版本，执行npm install就会升级到指定版本

    "dependencies": {
    "react": "^15.1.0",
    "react-native": "^0.27.2",

react-native upgrade：更新项目templates文件

react升级同理


## nodejs 升级

Chocolatey是一套可以用來在Windows上用命令列(powershell)安裝軟體套件的程式，
類似Mac OS X上的homebrew或ubuntu的apt-get。

更新Node命令：

choco upgrade nodejs



## node_modules文件名或扩展名太长如何删除 ##
步骤一：npm install -g rimraf
步骤二：rimraf node_modules

## 设定npm全局模块以及cache的路径

npm作为一个NodeJS的模块管理。需要先配置npm的全局模块的存放路径以及cache的路径。

例如讲两个文件夹放在NodeJS的主目录下。先在NodeJs安装目录下建立“node_global”及“node_cache”两个文件夹。

在cmd中键入两行命令：

npm config set prefix "D:\Program Files\nodejs\node_global"

和

npm config set cache "D:\Program Files\nodejs\node_cache"

然后在C:\Users\yourname\会看到生成.npmrc：

prefix=D:\Program Files\nodejs\node_global

cache=D:\Program Files\nodejs\node_cache


> [Windows环境下的NodeJS+NPM+Bower安装配置](http://jingyan.baidu.com/article/2d5afd69e243cc85a2e28efa.html)
> [Node.js各作業系統更新方式 ](http://eddychang.me/blog/javascript/58-nodes-update.html)