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

