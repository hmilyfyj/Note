---
title: 在Docker中配置时区
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

统一容器时区为东八区。

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