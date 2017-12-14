title: PHP 调试技巧 & 工具
date: 2016-04-14 22:07
tags: [PHP,折腾]
categories: PHP
---

陆续补充一些好用的工具。

目前有：

### 工具类

	1. var-dumper
	2. Xdebug
	3. Sentry（神器）
	4. Laravel5.5 最新版报错时会显示错误栈。

### 技巧类


<!-- more -->

---

# 技巧

# 工具

## Var-dumper

可以取代费眼的`var_dump` ，效果图：

![enter image description here](http://ww1.sinaimg.cn/large/c048f998jw1f2wl8udldnj20la09awfy.jpg)

### 安装

#### 项目中使用

    composer require symfony/var-dumper

#### 全局使用

首先，全局安装。

```
composer global require symfony/var-dumper;
```

修改`php.ini` ：

     auto_prepend_file = C:\Users\fengy\AppData\Roaming\Composer\vendor\autoload.php

从此所有PHP 脚本都可以使用 `composer` 全局安装的类库。


## Xdebug

### 安装

[访问官方文档](https://xdebug.org/docs/install)

