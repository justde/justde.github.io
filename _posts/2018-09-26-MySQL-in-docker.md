---
layout: post
title: '基于docker安装MySQL'
date: 2018-09-26
author: Justd
cover: '/assets/img/2018-9/09-26/ima.jpeg'
tags: MySQL docker  
---
为了更好的管理，打算把MySQL、redis等服务放在虚拟机中统一部署，这样不会因为这些服务的问题影响到系统本身。前段时间正好在看docker相关的内容，打算在虚拟机中通过docker来使用MySQL等服务。  
这次先记录安装MySQL的过程。  
# docker安装  
### 首先安装docker服务  
``` bash
yum -y install docker   
``` 
![](/assets/img/2018-9/09-26/installdocker.png)

### docker中搜索可用镜像
```
docker search mysql
```
![](/assets/img/2018-9/09-26/dockermysql.png)


# 拉取MySQL镜像
```
docker pull mysql:5.6
```
![](/assets/img/2018-9/09-26/installmysql.png)

### 查看MySQL镜像
```
docker image ls
```
![](/assets/img/2018-9/09-26/dockerimage.png)


# 运行MySQL
```
docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d -i -p 3306:3306 --restart=always  mysql:5.6
```
![](/assets/img/2018-9/09-26/run.png)
以上参数的含义：  
- --name mysql  将容器命名为mysql，后面可以用这个name进行容器的启动暂停等操作   
- -e MYSQL_ROOT_PASSWORD=123456 设置MySQL密码为123456
- -d 此容器在后台运行,并且返回容器的ID
- -i 以交互模式运行容器
- -p 进行端口映射，格式为`主机(宿主)端口:容器端口` 
- --restart=always 当docker重启时，该容器自动重启
# 进入MySQL容器
```
docker exec -ti mysql bash
```
![](/assets/img/2018-9/09-26/exec.png)


参考：[常见的docker命令](http://justd.xyz/2018/09/25/Common-docker-commands.html)