---
title: Linux服务器所见所闻
date: 2019-05-19 11:29:33
tags:
keywords:
description:
---

# docker使用遇到的一些问题

## docker报错：Cannot connect to the Docker daemon. Is the docker daemon running on this host?

<!-- more -->

docker这种报错一般情况都是docker未启动对于这种情况只用重启docker就行了：

service docker restart  

还有一种情况则是docker配置文件出错按照提示查看报错，并找到相应位置进行更改：
``` bash
systemctl status docker.service
```
或
``` bash
journalctl -xn
```
对于初学者且很难找到报错位置并改正的同学，推荐一个快速的方法---重装大法

``` bash
cd /var/lib/docker
rm –rf *
service docker restart
```

ps:如果在删除docker文件时报错删除不了，可以选择删除docker或者暂停docker  yum remove docker

## Docker启动Get Permission Denied

>问题描述

安装完docker后，执行docker相关命令，出现

``` bash
”Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/images/json: dial unix /var/run/docker.sock: connect: permission denied“
```

>解决方法1

使用sudo获取管理员权限，运行docker命令

>解决方法2

docker守护进程启动的时候，会默认赋予名字为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令

``` bash
sudo groupadd docker     #添加docker用户组
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
docker ps    #测试docker命令是否可以使用sudo正常使用
```
