---
title: Laravel 多用户认证
date:  2017-07-15 10:40:59
tags: [Laravel,php]
categories: Laravel
grammar_cjkRuby: true
---

在 `Laravel` 中用到鉴权方式有两种，分别是 `jwt` 和 `微信Oauth2`，那么问题来了，当我有多个微信公众号、多个用户表需要鉴权时，该如何配置呢？

<!-- more -->

---

### 一、Jwt 多用户认证
[Dingo/Api](https://github.com/dingo/api)
[Jwt-Auth](https://github.com/tymondesigns/jwt-auth)

这里需要了解一下 jwt 认证原理以及各个字段的含义

````
sub Subject - This holds the identifier for the token (defaults to user id)

iat Issued At - When the token was issued (unix timestamp)

exp Expiry - The token expiry date (unix timestamp)

nbf Not Before - The earliest point in time that the token can be used (unix timestamp)

iss Issuer - The issuer of the token (defaults to the request url)

jti JWT Id - A unique identifier for the token (md5 of the sub and iat claims)

aud Audience - The intended audience for the token (not required by default)
````

[更加详细的介绍看这里](https://github.com/tymondesigns/jwt-auth/wiki/Creating-Tokens)

#### 单用户认证

主要配置的地方是 Provider：

`auth.php` 配置：
````php
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\Members::class,
        ],
    ],
````

更多配置看这里：[Configuration](https://github.com/tymondesigns/jwt-auth/wiki/Configuration)


#### 多用户

1.修改 Provider


2.修改 Secrect

之前提到过，sub 作为用户标识，一般是用户的 id。为了避免多用户验证时 token “串号”，一定要记得修改 secrect

Laravel 配置：
修改 `Providers/RouteServiceProvider.php` 文件：
````php
                Config::set('auth.providers.users.model', Customer::class);
                Config::set('jwt.secret', env('JWT_SECRET_CUSTOMER'));
````

待解决：

为了动态修改 `auth.php` 和 `jwt.php` 中的配置，我选择了 Hack `Providers/RouteServiceProvider.php`  文件。应该有更优雅的解决办法。


