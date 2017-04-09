title: cors 部分情况下失效
date:  2017-04-09 7:52:41
tags: Laravel,踩坑
categories: Laravel
---

项目使用了 `dingo/api`，发现在抛出某些异常后没有自动支持 `cors`，跨域访问报错：
```javascript
XMLHttpRequest cannot load http://localhost:91/login. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:4200' is therefore not allowed access.
```
可能是 cors 中间件没有对所有异常做处理，解决办法，修改 `config/api.php`：

```php
<?php

return [

    'middleware' => [
        'palanik\lumen\Middleware\LumenCors'
    ]

];
```

https://github.com/dingo/api/issues/876
https://github.com/dingo/api/issues/930