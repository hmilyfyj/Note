---
title: 2017-2-13-composer-issue
grammar_cjkRuby: true
date: 2016-02-11 08:43
tags: [Composer,issue]
categories: Composer
---


## 无法安装

执行：
````shell
composer require ltsteam/oapi-taobao-client
````

报错：

````shell
  [InvalidArgumentException]
  Could not find package ltsteam/oapi-taobao-client at any version for your minimum-stability (stable). Check the pac
  kage spelling or your minimum-stability
````

解决：

````shell
composer require orzcc/taobao-top-client dev-master
````