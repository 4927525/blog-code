---
title: "docker 集群中 nginx 配置 php 需要注意的地方"
date: 2021-12-17T11:09:08+08:00
Description: ""
Tags: ["nginx", "php", "docker"]
Categories: ["docker"]
DisableComments: false
---



# 序

最近docker部署集群的时候遇到了一个问题。于是就踩坑了...

![src=http---b-ssl](https://cdn.jsdelivr.net/gh/4927525/images@master/20211217/src=http---b-ssl.duitang.com-uploads-item-201806-08-20180608010339_nUV5Y.thumb.400_0.jpeg&refer=http---b-ssl.duitang.3cfz68ibfo40.jpg)

## 问题复现

那就是php和nginx不在同一个容器，却要通过nginx配置的server_name和listen来访问php项目。

之前在本地集成环境部署的时候没有考虑过这个问题，是因为只要nginx的root根目录和php的root根目录指向同一个目录，这样把项目文件放到这个目录下就可同时访问，也就不存在说是php来访问还是nginx来访问的说法。

但是在集群中也就需要考虑这个问题了。

## 思路

静态文件需要在 nginx 容器内，php代码放在 php-fpm 的容器内，在nginx的配置里 php 部分设置php-fpm 所在容器内php的路径就可以了。

## 实现

```shell
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name api.com;

    location /     {
        # 隐藏index.php
        if (!-e $request_filename) {
            rewrite ^/(.*)$ /index.php?$1 last;
        }
        index index.html index.htm index.php;
        autoindex on;
    }
    location ~ \.php$ {
        # php项目
        root /var/www/html/phalapi/api/public;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

![src=http---c-ssl](https://cdn.jsdelivr.net/gh/4927525/images@master/20211217/src=http---c-ssl.duitang.com-uploads-item-202004-06-20200406031436_YdLWc.thumb.400_0.gif&refer=http---c-ssl.duitang.6iau48gki5g0.gif)
