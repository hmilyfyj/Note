---
title: 2018-12-15-为 PHP Docker 镜像添加扩展
date: 2018-12-15 12:22:42
tags: Docker
categories: Docker
---


<!-- more -->

---
```
apt-get update

apt-get install -y libwebp-dev libjpeg62-turbo-dev libpng-dev libxpm-dev  libfreetype6-dev

docker-php-ext-configure gd --with-gd --with-webp-dir --with-jpeg-dir  --with-png-dir --with-zlib-dir --with-xpm-dir --with-freetype-dir --enable-gd-native-ttf

docker-php-ext-install gd
```