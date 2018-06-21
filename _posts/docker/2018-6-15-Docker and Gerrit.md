---
layout: post
title: Docker & Gerrit 搭建Git服务器
date: 2018-5-15
excerpt: "Docker & Gerrit 搭建Git服务器"
categories: 系统
tags: [系统]
comments: true
---

* content
{:toc}



# 简介

使用[Docker](https://www.docker.com/) & Gerrit 搭建Git服务器

Docker： 开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。可以安装任何的软件或系统

常用的git管理工具：gitolite ,gitosis ,gerrit , gitlib 知乎

# 1. 安装Docker

sudo apt-get install docker.io

# 2.导入image

导入docker镜像，命令：docker load < gerrit_v1.4.tar

# 3. 查看所有装载的image

docker images

# 4. 启动image

新增容器命令：docker run -p 29418:29418 -p 8881:80 -it gerrit:1.4 /bin/bash （每次产生一个新的容器）

- 修改docker下的/etc/apache2/sites-available/000-default.conf文件，将serverName修改成正确的ip
- 修改docker下的/usr/soft/gerrit/etc/gerrit.config文件，将url修改成正确的ip
- 重启docker下的apache2，命令：/etc/init.d/apache2 restart
- 重启docker下的gerrit，命令：/usr/soft/gerrit/bin/gerrit.sh restart

docker ps -a 查看d00是否存在

启动 docker start d00

进入 sudo docker exec -it d00 /bin/bash  


# Reference

[Docker菜鸟教程](http://www.runoob.com/docker/docker-tutorial.html)

[Docker image文件下载：Explore Official Repositories](https://hub.docker.com/explore/)





