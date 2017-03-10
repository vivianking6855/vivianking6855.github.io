---
layout: post
title: ReactNative Windows下npm设定小结
date: 2016-05-25
excerpt: "Windows下npm设定小结"
categories: ReactNative
tags: [ReactNative]
comments: true
---

* content
{:toc}


# 1. npm 安装

npm install --save libname (from network)
npm install --save "E:\Download\react-native-router-master" (from local)

# 2. 设定npm全局模块以及cache的路径

npm作为一个NodeJS的模块管理。需要先配置npm的全局模块的存放路径以及cache的路径。

例如讲两个文件夹放在NodeJS的主目录下。先在NodeJs安装目录下建立“node_global”及“node_cache”两个文件夹。

在cmd中键入两行命令：

npm config set prefix "D:\Program Files\nodejs\node_global"

和

npm config set cache "D:\Program Files\nodejs\node_cache"

然后在C:\Users\yourname\会看到生成.npmrc：

prefix=D:\Program Files\nodejs\node_global

cache=D:\Program Files\nodejs\node_cache

# 3. npm升级插件

npm可以很方便的升级各种插件。例如升级express框架命令

- npm update express （非全局安装）
- npm update -g express （express全局安装）

也可以使用安装命令来重装，在这里是等效于update：npm install -g express

## 3.1 升级npm自身
1. 先cd到nodejs安装目录：（例如： D:\Program Files\nodejs）

    cd "D:\Program Files\nodejs"

2. npm update npm

## 3.2 npm 常用命令
1. npm info react-native
2. npm update -g react-native-cli：更新cli
3. 查看用户配置文件：过npm config get userconfig
3. 查看全局配置文件： npm config get globalconfig 
4. npm config list 列出config

## 3.3 升级react-native

### 3.3.1 升级react-native和项目templates文件

可参照上面的升级所有dependencies，也可以：
1. npm info react-native
2. 修改package.json中的依赖版本，执行npm install就会升级到指定版本
3. 更新template中的android，ios template文件：react-native upgrade

### 3.3.2 升级所有项目dependencies到最新版本
1. 使用npm -g outdated 查看 dependencies是否outdate
2. package.json里面update version到最新，然后npm install就全部升级了


# 4. 管理npm registries工具：[nrm](https://www.npmjs.com/package/nrm)

管理npm registries,工具

安装：npm install -g nrm

# 5. nodejs 升级

直接从[nodejs官网](https://nodejs.org/en/)下载

# 6. npm 翻墙

npm翻墙提速
1. 关闭npm的https

    npm config set strict-ssl false

2. 设置npm的获取地址

    npm config set registry "http://registry.npmjs.org/"

3. 如果有代理服务器可以使用proxy代理方式。设置npm获取的代理服务器地址：

    npm config set proxy=http://代理服务器ip:代理服务器端口

4. 清除npm的代理命令如下：

    npm config delete http-proxy
    npm config delete https-proxy

# Tips

1. 删除路径过长文件夹，例如node_modules. 可以用rimraf （安装：npm install rimraf -g)
2. 官方推荐的yarn(Facebook发布的新的node.js包管理器Yarn替代npm）
 
    安装：npm install -g yarn
    
    更快速和效率的管理nodejs包


> [Windows环境下的NodeJS+NPM+Bower安装配置](http://jingyan.baidu.com/article/2d5afd69e243cc85a2e28efa.html)

> [Node.js各作業系統更新方式 ](http://eddychang.me/blog/javascript/58-nodes-update.html)

> [npm升级所有可更新包](http://www.tuicool.com/articles/UbyY7rY)

> [NPM设置代理](https://my.oschina.net/deathdealer/blog/208919)

> [npm国内被墙的解决方法](http://snoopyxdy.blog.163.com/blog/static/60117440201422695653698/) 