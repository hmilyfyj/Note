---
title: Nginx 下禁止IP直连
date:  2017-08-31 11:33:52
tags: Nginx,Secure
categories: Note
grammar_cjkRuby: true
---



<!-- more -->

---

``` nginx
server {
                listen 80 default;
                server_name _;
                return 500;
}
server {
                listen 443 default;
                server_name _;
                ssl_certificate      ssl.crt;
                ssl_certificate_key  ssl.key;
                return 500;
 }
```


参考资料
https://www.muzifei.com/post/deny-access-via-ip.html