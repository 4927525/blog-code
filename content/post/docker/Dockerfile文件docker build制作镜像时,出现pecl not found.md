---
title: "Dockerfile文件docker build制作镜像时,出现pecl not found"
date: 2021-12-14T09:49:43+08:00
Description: ""
Tags: ["docker", "Dockerfile"]
Categories: ["docker"]
DisableComments: false
---


问题复现：
官方给出的扩展安装地址https://github.com/hyperf/hyperf-docker
```shell
RUN apk add --no-cache librdkafka-dev \
&& pecl install rdkafka \
&& echo "extension=rdkafka.so" > /etc/php7/conf.d/rdkafka.ini
```
以上代码加入到Dockerfile中，重新build会报错
```bash
/bin/sh: pecl: not found
```

解决方法：
```shell
在https://github.com/codecasts/php-alpine/tree/master/scripts/v3.12/php-7.4找到对应的扩展，直接
将目录名加入到Dockerfile中解决
```

```shell
RUN set -ex \
    && apk add php7-mongodb
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/87c16c4900cb49cfa50f0050cf66281b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBANDkyNzUyNQ==,size_20,color_FFFFFF,t_70,g_se,x_16)


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

EXPOSE 9401

ENTRYPOINT ["php", "/opt/www/bin/hyperf.php", "start"]

```
