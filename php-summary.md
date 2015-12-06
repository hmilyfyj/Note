
title: PHP知识点
date: 2015-12-6 22:27:3
tags: [php,ip]
categories: php
---

### 踩过的坑

#### `Using $this when not in object context in`

静态方法内无法使用$this,解决方法：

    self::method();

[参考地址](http://blog.csdn.net/yageeart/article/details/6662059)

### 小TIPS

#### 获取物理路径

    realpath();



