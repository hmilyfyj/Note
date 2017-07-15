---
title: Laravel 队列问题
date:  2017-07-13 9:45:18
tags: [shell]
categories: Note
grammar_cjkRuby: true
---

推送队列到线上，出现如下报错：

````
Symfony\Component\Debug\Exception\FatalErrorException
method_exists(): The script tried to execute a method or access a property of an incomplete object. Please ensure that the class definition "App\Jobs\Taobao\Order\ProcessFixBaichuanBuyerId" of the object you are trying to operate on was loaded _before_ unserialize() gets called or provide a __autoload() function to load the class definition
````

### 原因
1. 多个 Laravel 公用一个队列
2. 修改了当前项目地址（软连接）

导致：即将运行的队列不存在

### 解决
重启队列即可。

````shell
supervisorctl reload
````