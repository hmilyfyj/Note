title: 配置Docker镜像的时区
date: 2018-01-25 09:29:15
tags: [Docker,运维]
categories: Docker
---

统一镜像时区为东八区。

<!-- more -->

## debian
``` dockerfile
RUN mkdir -p /var/log/php-fpm && export TZ=Asia/Shanghai && echo $TZ | tee /etc/timezone && dpkg-reconfigure --frontend noninteractive tzdata
```

## alpine

``` dockerfile
RUN apk update && apk add ca-certificates && \
    apk add tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone
```
## ubuntu


## 参考资料
https://www.zybuluo.com/zwh8800/note/337111
