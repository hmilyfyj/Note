---
title: Laravel 队列与软连接的兼容问题
grammar_cjkRuby: true
tags: Laravel,PHP
categories: Laravel
---

待填坑

<!-- more -->

---

背景：每次部署代码后需要重启队列。但是现有的状况下，重启队列会导致正在运行的队列终端，可能出现不可逆的损害。

```
php artian queue:work
php artian queue:listen
```