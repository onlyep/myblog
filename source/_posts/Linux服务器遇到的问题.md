---
title: Linux服务器遇到的问题
date: 2019-05-19 11:29:33
tags: Linux
keywords: Linux
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

# nginx 配置

## 启动
``` bash
[root@LinuxServer sbin]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```
## 重启

>1、验证nginx配置文件是否正确

方法一：进入nginx安装目录sbin下，输入命令./nginx -t
看到如下显示nginx.conf syntax is ok
nginx.conf test is successful
说明配置文件正确！
方法二：在启动命令-c前加-t

>2、重启Nginx服务

方法一：进入nginx可执行目录sbin下，输入命令./nginx -s reload 即可
方法二：查找当前nginx进程号，然后输入命令：kill -HUP 进程号 实现重启nginx服务

## Nginx.conf 配置

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
  worker_connections  1024;
}

http {
  include       mime.types;
  default_type  application/octet-stream;

  #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
  #                  '$status $body_bytes_sent "$http_referer" '
  #                  '"$http_user_agent" "$http_x_forwarded_for"';

  #access_log  logs/access.log  main;

  sendfile        on;
  #tcp_nopush     on;

  #keepalive_timeout  0;
  keepalive_timeout  65;

  #gzip  on;

  server {
    listen       80;
    server_name www.varbug.top;# 这里配置我们的域名，确定域名已解析到本机

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
      root   html;
      index  index.html index.htm;
    }
    location ^~ /jianshu/ {
      alias   /html/jianshu/;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
      root   html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
  }

  server {

		listen 80;

		server_name cloud.varbug.top;

		location / {
      proxy_redirect off;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://127.0.0.1:8181; #转发到本机8181端口 oneindex项目
	  }
	}

  # another virtual host using mix of IP-, name-, and port-based configuration
  #
  #server {
  #    listen       8000;
  #    listen       somename:8080;
  #    server_name  somename  alias  another.alias;

  #    location / {
  #        root   html;
  #        index  index.html index.htm;
  #    }
  #}


  # HTTPS server
  #
  #server {
  #    listen       443 ssl;
  #    server_name  localhost;

  #    ssl_certificate      cert.pem;
  #    ssl_certificate_key  cert.key;

  #    ssl_session_cache    shared:SSL:1m;
  #    ssl_session_timeout  5m;

  #    ssl_ciphers  HIGH:!aNULL:!MD5;
  #    ssl_prefer_server_ciphers  on;

  #    location / {
  #        root   html;
  #        index  index.html index.htm;
  #    }
  #}
}
```
