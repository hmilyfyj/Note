title: Laravel 笔记
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


## 路由

    Route::get($uri, $callback);
    Route::match(['get', 'post'], '/', function () {
    //
	});

	Route::any('foo', function () {
	    //
	});

### 带参数

	 Route::get('user/{id}', function ($id) {
	    return 'User '.$id;
	});


	Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
	    //
	});

	//可选参数
	Route::get('user/{name?}', function ($name = null) {
	    return $name;
	});

	Route::get('user/{name?}', function ($name = 'John') {
	    return $name;
	});


**注：注意：路由参数不能包含 - 字符，需要的话可以使用 _ 替代。**


### 路由别名

	Route::get('user/profile', ['as' => 'profile', function () {
	    //
	}]);

	Route::get('user/profile', [
	    'as' => 'profile', 'uses' => 'UserController@showProfile'
	]);

	Route::get('user/profile', 'UserController@showProfile')->name('profile');

	//通过在路由群组的属性数组中指定 as 关键字来为群组中的路由设置一个共用的路由名前缀
	Route::group(['as' => 'admin::'], function () {
	    Route::get('dashboard', ['as' => 'dashboard', function () {
	        // 路由被命名为 "admin::dashboard"
	    }]);
	});

	//为起了别名的路由生成Url并跳转
	$url = route('profile');
	$redirect = redirect()->route('profile');

	//给命名路由传参
	Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
	     //
	}]);
	$url = route('profile', ['id' => 1]);

### 群组路由

>路由群组允许我们在多个路由中共享路由属性，比如中间件和命名空间等。共享属性以数组的形式作为第一个参数被传递给 Route::group 方法。

#### 使用中间件

	Route::group(['middleware' => 'auth'], function () {
	    Route::get('/', function () {
	        // 使用 Auth 中间件
	    });

	    Route::get('user/profile', function () {
	        // 使用 Auth 中间件
	    });
	});


#### 命名空间

	Route::group(['namespace' => 'Admin'], function(){
	    // 控制器在 "App\Http\Controllers\Admin" 命名空间下

	    Route::group(['namespace' => 'User'], function(){
	        // 控制器在 "App\Http\Controllers\Admin\User" 命名空间下
	    });
	});

>默认情况下，RouteServiceProvider 引入 routes.php 并指定其下所有控制器类所在的默认命名空间App\Http\Controllers，因此，我们在定义的时候只需要指定命名空间 App\Http\Controllers 之后的部分即可。

#### 前缀

	Route::group(['prefix' => 'admin'], function () {
	    Route::get('users', function () {
	        // 匹配 "/admin/users" URL
	    });
	});

	Route::group(['prefix' => 'accounts/{account_id}'], function () {
	    Route::get('detail', function ($account_id) {
	        // 匹配 accounts/{account_id}/detail URL
	    });
	});


如果路由规则重复，则使用先定义的。


### CSRF

>不需要自己编写代码去验证 POST、PUT 或者 DELETE 请求的 CSRF 令牌，因为 Laravel 自带的 HTTP 中间件VerifyCsrfToken 会为我们做这项工作：将请求中输入的 token 值和 Session 中的存储的 token 作对比来进行验证。


可以使用帮助函数 csrf_field 来实现：

	<?php echo csrf_field(); ?>

辅助函数 csrf_field 会生成如下 HTML：

	<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

当然还可以使用 Blade 模板引擎提供的方式：

	{!! csrf_field() !!}

#### CSRF保护排除

	<?php

	namespace App\Http\Middleware;

	use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

	class VerifyCsrfToken extends BaseVerifier
	{
	    /**
	     *从CSRF验证中排除的URL
	     *
	     * @var array
	     */
	    protected $except = [
	        'stripe/*',
	    ];
	}

### 路由绑定

#### 隐式绑定

	Route::get('api/users/{user}', function (App\User $user) {
	    return $user->email;
	});

>在这个例子中，由于类型声明了 Eloquent 模型 App\User，对应的变量名 $user 会匹配路由片段中的{user}，这样，Laravel 会自动注入与请求 URI 中传入的 ID 对应的用户模型实例。

##### 修改绑定

重写 Eloquent 模型类的 getRouteKeyName 方法：

	/**
	 * Get the route key for the model.
	 *
	 * @return string
	 */
	public function getRouteKeyName()
	{
	    return 'slug';
	}













