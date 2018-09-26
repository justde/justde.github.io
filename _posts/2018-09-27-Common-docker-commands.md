---
layout: post
title: '常用的docker命令'
date: 2018-09-27
author: Justd
cover: '/assets/img/2018-9/09-26/ima.jpeg'
tags:  docker  
---
## 简单的记录常用的docker命令 
> docker version  

查看docker版本

>docker image pull helloworld

拉取名为 `helloword` 的镜像  
>docker images ls    

查看docker镜像    
 
>docker rmi <CONTAINER ID>

根据id删除镜像

>docker container run hello-world

运行image文件（执行docker run时，已经包含了该步骤）
>docker ps / docker ps --all

查看运行中/任意状态的容器
>docker container ls

查看容器
>docker container kill <CONTAINER ID>

杀掉正在运行的容器
>docker container rm <CONTAINER ID>

删除指定容器
>docker logs <CONTAINER ID>

查看容器运行日志
>docker update --restart=always <CONTAINER ID>

已经启动的镜像，设置docker的重启时，同时重启该容器