---
title: 2017-7-13-Laravel-Issues
date:  2017-07-13 9:45:18
tags: Note
categories: Note
grammar_cjkRuby: true
---

推送队列到线上，出现如下报错：

````
Symfony\Component\Debug\Exception\FatalErrorException
method_exists(): The script tried to execute a method or access a property of an incomplete object. Please ensure that the class definition "App\Jobs\Taobao\Order\ProcessFixBaichuanBuyerId" of the object you are trying to operate on was loaded _before_ unserialize() gets called or provide a __autoload() function to load the class definition
````

### 解决