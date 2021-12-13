---
title: "docker从零部署项目包括不间断线上更新"
date: 2021-12-13T14:15:56+08:00
Description: ""
Tags: ["docker", "docker-compose", "Dockerfile"]
Categories: ["docker"]
DisableComments: false
---

## 序

记录自己用Docker从零搭建项目部署到服务器和使用红黑部署来保证程序不中断更新的心得和经验。

> 对Docker还不熟悉的小伙伴可以看下这篇文章
> 
> [Docker最新超详细版教程通俗易懂]: https://blog.csdn.net/hzbskak/article/details/118367706?spm=1001.2014.3001.5501

![](https://img0.baidu.com/it/u=654394651,2129990857&fm=26&fmt=auto)

## 1. 编写项目微服务

这是前提工作，必须确保项目是可以运行的。

![](https://img0.baidu.com/it/u=2759264495,1166948461&fm=26&fmt=auto)

## 2. 创建网络

这是容器通信的基础

```bash
$ docker network create pay-notify-net
```

## 3. Dockerfile构建镜像

### 3.1. 编写一个dockerfile文件

在项目根目录下新建一个Dockerfile文件，用来构建镜像

```shell
# Default Dockerfile
#
# @link     https://www.hyperf.io
# @document https://hyperf.wiki
# @contact  group@hyperf.io
# @license  https://github.com/hyperf/hyperf/blob/master/LICENSE

FROM hyperf/hyperf:7.4-alpine-v3.11-swoole
LABEL maintainer="Hyperf Developers <group@hyperf.io>" version="1.0" license="MIT" app.name="Hyperf"

##
# ---------- env settings ----------
##
# --build-arg timezone=Asia/Shanghai
ARG timezone

ENV TIMEZONE=${timezone:-"Asia/Shanghai"} \
    APP_ENV=prod \
    SCAN_CACHEABLE=(true)

# update
RUN set -ex \
    # show php version and extensions
    && php -v \
    && php -m \
    && php --ri swoole \
    #  ---------- some config ----------
    && cd /etc/php7 \
    # - config PHP
    && { \
        echo "upload_max_filesize=128M"; \
        echo "post_max_size=128M"; \
        echo "memory_limit=1G"; \
        echo "date.timezone=${TIMEZONE}"; \
    } | tee conf.d/99_overrides.ini \
    # - config timezone
    && ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    # ---------- clear works ----------
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"

RUN set -ex \
    && apk add php7-mongodb

WORKDIR /opt/www

# Composer Cache
# COPY ./composer.* /opt/www/
# RUN composer install --no-dev --no-scripts

COPY . /opt/www
RUN composer install --no-dev -o && php bin/hyperf.php

EXPOSE 9501

ENTRYPOINT ["php", "/opt/www/bin/hyperf.php", "start"]
```

### 3.2. docker build 构建成为一个镜像

```bash
# -t 打标签
$ docker build -t pay-notify:1.0.1 .

[+] Building 3.9s (11/11) FINISHED
 => [internal] load build definition from Dockerfile                                                                                               0.0s
 => => transferring dockerfile: 32B                                                                                                                0.0s
 => [internal] load .dockerignore                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                    0.0s
 => [internal] load metadata for docker.io/hyperf/hyperf:7.4-alpine-v3.11-swoole                                                                   0.0s
 => [1/6] FROM docker.io/hyperf/hyperf:7.4-alpine-v3.11-swoole                                                                                     0.0s
 => [internal] load build context                                                                                                                  0.3s
 => => transferring context: 573.27kB                                                                                                              0.3s
 => CACHED [2/6] RUN set -ex     && php -v     && php -m     && php --ri swoole     && cd /etc/php7     && {         echo "upload_max_filesize=12  0.0s
 => CACHED [3/6] RUN set -ex     && apk add php7-mongodb                                                                                           0.0s
 => CACHED [4/6] WORKDIR /opt/www                                                                                                                  0.0s
 => [5/6] COPY . /opt/www                                                                                                                          0.5s
 => [6/6] RUN composer install --no-dev -o && php bin/hyperf.php                                                                                   2.3s
 => exporting to image                                                                                                                             0.6s
 => => exporting layers                                                                                                                            0.6s
 => => writing image sha256:d81127be6073d3cb76c2323ad5a8c58fb5d9372b805e00310f0cdf9dc80b6f4d                                                       0.0s
 => => naming to pay-notify:1.0.1
```

## 4. 推送到私有仓库

### 4.1 给本地镜像打标签

> 启动私有仓库官网地址：
> 
> [Deploy a registry server]: https://docs.docker.com/registry/deploying/

```bash
$ docker tag pay-notify:1.0.1 xxx.com/pay-notify:1.0.1
```

### 4.2 登录私有仓库

```bash
$ docker login xxx.com
Authenticating with existing credentials...
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### 4.3 推送私有仓库

这里要将本地镜像推送到私有仓库中，方便服务器拉取。

```bash
$  docker push xxx.com/pay-notify:1.0.1
The push refers to repository [xxx.com/pay-notify]
ff4a8d479433: Pushed
13d5b39dde0c: Pushed
79ced1246ab1: Layer already exists
7b6bce294554: Layer already exists
6df89ea81a26: Layer already exists
cfd4b1189b9e: Layer already exists
04ab725ccb75: Layer already exists
55bf8cdb863f: Layer already exists
39982b2a789a: Layer already exists
1.0.7: digest: sha256:3026839811d8d4cd125d58575120026a132b7747beda4c7813aa8e26d05543dd size: 2204
```

### 4.4 获取私有仓库list

```bash
curl -X GET https://xxx.com/v2/pay-notify/tags/list # 或者浏览器访问
{
    name: "pay-notify",
    tags: [
        "1.0.2",
        "1.0.4",
        "1.0.7",
        "1.0.1",
        "1.0.6",
        "1.0.3",
        "1.0.5"
    ]
}
```

## 5. 编排Docker-compose

### 5.1 yaml文件编写

将pay_notify1.yml文件上传到服务器上

```yaml
version: '3.3'
services:
  hc:
    image: xxx.com/pay-notify:1.0.1
    ports:
      - 9501:9501
    environment:
      - "APPENV=prod"
      - "REDIS_HOST=pay_notify_redis_1"
      - "DB_HOST=xxxx"
      - "DB_DATABASE=xxx"
      - "DB_USERNAME=xxx"
      - "DB_PASSWORD=xxx"
      - "SYS_DEBUG=false"
    networks:
      - pay-notify-net
    depends_on:
      - redis
  redis:
    image: redis:6.0.10
    volumes:
      - /www/wwwroot/test/redis/data:/data
      - /www/wwwroot/test/redis/redis.conf:/usr/local/etc/redis/redis.conf
      - /www/wwwroot/test/redis/logs:/logs
    ports:
      - 6379:6379
    networks:
      - pay-notify-net
networks:
  pay-notify-net:
    external: true
```

### 5.2 启动项目

```bash
$ docker-compose -f pay_notify1.yml -p pay-notify up -d 

Pulling hc2 (xxx.com/pay-notify:1.0.1)...
1.0.1: Pulling from pay-notify
6a428f9f83b0: Already exists
069b9d0dbac2: Already exists
0be19dcd9b08: Already exists
720818493b30: Already exists
11053722a517: Already exists
dc75e73f3321: Already exists
a121be122369: Already exists
9cacf81feb94: Pull complete
409c206cf402: Pull complete
Digest: sha256:b636a24f4101912bac3639a83aff3453fedfd5b8f904340707d97e04e6ac7591
Status: Downloaded newer image for xxx.com/pay-notify:1.0.1
Recreating pay_notify_hc_1 ... done
Recreating pay_notify_redis_1 ... done
```

### 5.3 测试

```bash
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED       STATUS       PORTS                    NAMES
2531c6e84038   xxx.com/pay-notify:1.0.1   "php /opt/www/bin/hy…"   3 hours ago   Up 3 hours   0.0.0.0:9501->9501/tcp   pay_notify_hc_1
bbdd7e15c9df   redis:6.0.10                           "docker-entrypoint.s…"   6 hours ago   Up 4 hours   0.0.0.0:6378->6379/tcp   pay_notify_redis_1
```

![](https://img0.baidu.com/it/u=1623228430,4099085262&fm=26&fmt=auto)

> 到这里其实就可以用Docker部署一个完整的项目了。
> 
> 如果想要完成线上不停机更新的话，就要用过反代来实现了。
> 
> 以下过程采用红黑部署完成

## 6.红黑部署

### 6.1 利用nginx反代实现红黑部署

利用反代映射两个端口，让这两个端口都可以请求项目

```shell
upstream hc { 
      server 127.0.0.1:9501; 
      server 127.0.0.1:9502; 
}
server
{

    location /pay_notify/ {
      # Do not allow connections from docker 1.5 and earlier
      # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
      if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
        return 404;
      }

      # To add basic authentication to v2 use auth_basic setting.
      #auth_basic "Registry realm";
      #auth_basic_user_file /www/wwwroot/auth/docker_registry.password;

      proxy_pass                          http://hc/pay/notify;
      proxy_set_header  Host              $http_host;   # required for docker client's sake
      proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
      proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header  X-Forwarded-Proto $scheme;
      proxy_read_timeout                  900;
}
```

### 6.2 平滑更新版本

过了一段时间我们要发布pay-notify:1.0.2版本。

此时我们还是通过3、4、5来进行构建、打包、发布、运行。

因为线上正在运行的容器占用了9501这个端口，此时我们就要用到nginx反代的另一个端口来进行映射。

```shell
version: '3.3'
services:
  hc2: # 服务名
    image: xxx.com/pay-notify:1.0.2 # 版本号发生变化
    ports:
      - 9502:9501 # export端口发生变化
    environment:
      - "APPENV=prod"
      - "REDIS_HOST=pay_notify_redis_1"
      - "DB_HOST=xx"
      - "DB_DATABASE=xxx"
      - "DB_USERNAME=xxx"
      - "DB_PASSWORD=xxx"
      - "SYS_DEBUG=false"
    networks:
      - pay-notify-net
    depends_on:
      - redis
  redis:
    image: redis:6.0.10
    volumes:
      - /www/wwwroot/test/redis/data:/data
      - /www/wwwroot/test/redis/redis.conf:/usr/local/etc/redis/redis.conf
      - /www/wwwroot/test/redis/logs:/logs
    ports:
      - 6378:6379
    networks:
      - pay-notify-net
networks:
  pay-notify-net:
    external: true
```

这是两个容器都可以在后台运行

```shell
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED          STATUS          PORTS                    NAMES
51657b114757   xxx/pay-notify:1.0.1   "php /opt/www/bin/hy…"   21 minutes ago   Up 21 minutes   0.0.0.0:9501->9501/tcp   pay_notify_hc_1
01ca94858e7d   xxx/pay-notify:1.0.2   "php /opt/www/bin/hy…"   1 minutes ago   Up 1 minutes   0.0.0.0:9502->9501/tcp   pay_notify_hc2_1
bbdd7e15c9df   redis:6.0.10                           "docker-entrypoint.s…"   3 hours ago      Up 34 minutes   0.0.0.0:6378->6379/tcp   pay_notify_redis_1
```

此时我们可以停掉pay-notify:1.0.1让pay-notify:1.0.2在线上运行

```bash
$ docker stop pay_notify_hc_1
pay_notify_hc_1
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED          STATUS          PORTS                    NAMES
01ca94858e7d   xxx/pay-notify:1.0.2   "php /opt/www/bin/hy…"   1 minutes ago   Up 1 minutes   0.0.0.0:9502->9501/tcp   pay_notify_hc2_1
bbdd7e15c9df   redis:6.0.10                           "docker-entrypoint.s…"   3 hours ago      Up 34 minutes   0.0.0.0:6378->6379/tcp   pay_notify_redis_1
```

## 至此Docker部署全部完成。

一路上还是遇到了很多坑。还行最后都解决掉了。