---
title: 2018-7-3-PHP-Idempotency
date: 2018-07-03 13:38:40
tags: Nginx
categories: Nginx
---

HTTP 幂等方法，是指无论调用这个url多少次，都不会有不同的结果的HTTP方法。

<!-- more -->

---


https://www.cnblogs.com/weidagang2046/archive/2011/06/04/idempotence.html
https://laravel.com/api/5.1/Illuminate/Database/Eloquent/ModelNotFoundException.html
https://www.toptal.com/laravel/restful-laravel-api-tutorial
https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html
https://sofish.github.io/restcookbook/http%20methods/idempotency/

看到了一些讨论：

有些观点会觉得 幂等性不约束返回结果，仅约束 后端服务、资源的状态。

https://developer.mozilla.org/zh-CN/docs/Glossary/%E5%B9%82%E7%AD%89
https://stackoverflow.com/questions/6439416/deleting-a-resource-using-http-delete/45194747#45194747
https://stackoverflow.com/questions/4088350/is-rest-delete-really-idempotent

http://leedavis81.github.io/is-a-http-delete-requests-idempotent/