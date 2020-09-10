---
title: PHPUnit 和 Dusk
date: 2020-09-10 19:46:53
tags: PHP,PHPUnit,Docker
categories: PHP
---

## 初始化

Linux 中需要

# 接入 Dusk Dashboard
## 安装
安装时比较消耗 CPU，如果出现 Killed 时，考虑提高 CPU 限制。

## 运行
执行`php artisan dusk:dashboard`时出现如下报错：

```
stream_set_blocking() expects parameter 1 to be resource, null given
```

是因为方法被禁用了，调整 php.ini 文件的中的`disable_functions`即可。
## 访问
##### 直接暴露端口到宿主机
##### nginx 反向代理，通过 80 端口访问。

dashboard 后端出于安全考虑会检测 host，所以反向代理时需要指定`Host`


```nginx
server {
    listen   80;
    server_name private-dusk.local.tbxzs.cn;
    
    location /socket {
        proxy_pass http://__DOCKER_PHP_FPM_74__:9773/socket;
        # this magic is needed for WebSocket
        proxy_http_version  1.1;
        proxy_set_header    Upgrade $http_upgrade;
        proxy_set_header    Connection "upgrade";
        proxy_set_header    Host "private.local.tbxzs.cn:9773";
        proxy_set_header    X-Real-IP $remote_addr;
    }
    
    location / {
          proxy_pass http://__DOCKER_PHP_FPM_74__:9773;
          proxy_set_header    Host "private.local.tbxzs.cn:9773";
     }
}
```