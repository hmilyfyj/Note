---
title: Laravel 队列问题
date:  2017-07-13 9:45:18
tags: [Shell,Laravel,反省]
categories: Laravel
grammar_cjkRuby: true
---

代码到线上后发现除了问题，迅速手动修改了线上项目的软连接到上一版本。于是出现了如下问题。

<!-- more -->

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


### 总结
以上问题本可以在在测时阶段发现的。但并没有写测试用力。好好反省。