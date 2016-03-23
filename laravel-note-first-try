title: Laravel 笔记 - 部署
date: 2016-03-22 18:44
tags: [Laravel]
categories: Laravel
---

开始学习为 WEB 艺术家创造的 PHP 框架。

<!-- more -->

---

# 待了解

PSR-4


# 安装

## Composer

### 采用国内源

通过执行`composer config -l -g `找到 config.json文件的路径。

修改 config.json内容如下:

```javascript
"repositories": {
		"packagist": {
			"type": "composer",
			"url": "https://packagist.phpcomposer.com"
		}
	}
```


## Laravel

    composer global require "laravel/installer"

>请确保 PATH 环境变量已经添加了 `~/.composer/vendor/bin` 目录，这样，可执行文件 `laravel` 就能被你的系统检测到了。

# 使用

## 新建一个站点

    laravel new blog

## 基本配置

### 美化链接

#### Apache

修改`public/.htaccess`文件。

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

#### Nginx

	location / {
	    try_files $uri $uri/ /index.php?$query_string;
	}

### 环境

    .env 文件中的所有变量都会被加载到 PHP 的 $_ENV 超全局变量中。

### 配置缓存

为了提升应用程序的执行速度，在应用正式上线后，应通过 Artisan 的 config:cache 命令将所有配置文件合并到一个文件中并缓存起来。即

    php artisan config:cache

### 获取、修改配置

    $value = config('app.timezone');
    config(['app.timezone' => 'America/Chicago']);

### 维护

	php artisan down
	php artisan up

维护模式时响应请求的默认模板文件位于 `resources/views/errors/503.blade.php`

# 编码

## 流程

把这个放在最前面，因为一头扎进文档会感觉一头雾水，被牵着鼻子走。

### public/index.php  所有请求的入口

>index.php 文件载入 Composer 生成的自动加载设置，然后从 bootstrap/app.php 脚本获取 Laravel 应用实例，Laravel 的第一个动作就是创建服务容器实例

### HTTP/Console 内核 请求被发送至此

HTTP 内核：app/Http/Kernel.php

>HTTP 内核继承自 Illuminate\Foundation\Http\Kernel 类，该类定义了一个 bootstrappers 数组，这个数组中的类在请求被执行前运行，这些 内核启动过程中最重要的动作之一就是为应用载入服务提供者，应用的所有服务提供者都被配置在 config/app.php 配置文件的  providers 数组中。首先，所有提供者的 register 方法被调用，然后，所有提供者被注册之后，boot 方法被调用。

>服务提供者负责启动框架的所有各种各样的组件，比如数据库、队列、验证器，以及路由组件等，正是因为他们启动并配置了框架提供的所有特性，服务提供者是整个 Laravel 启动过程中最重要的部分。

 >配置了错误处理、日志、检测应用环境以及其它在请求被处理前需要执行的任务。




一旦应用被启动并且所有的服务提供者被注册，Request 将会被交给路由器进行分发，路由器将会分发请求到路由或控制器，同时运行所有路由指定的中间件。

### 模版

存放在`resources/views`






