title: Laravel 笔记 - 中间件
date: 2016-03-23 19:39
tags: [Laravel]
categories: Laravel
---

middleware ，在我看来类似于CI的hook

<!-- more -->

---

>理解中间件的最好方式就是将中间件看做 HTTP 请求到达目标动作之前必须经过的“层”，每一层都会检查请求并且可以完全拒绝它。


# 创建 

	//在 app/Http/Middleware 目录下创建一个新的中间件类 OldMiddleware，在这个中间件中，我们只允许提供的 age 大于 200 的访问路由，否则，我们将用户重定向到主页
	php artisan make:middleware OldMiddleware

	//如果 age<=200，中间件会返回一个 HTTP 重定向到客户端；否则，请求会被传递下去。将请求往下传递可以通过调用回调函数 $next 并传入  $request。
	<?php

	namespace App\Http\Middleware;

	use Closure;

	class OldMiddleware
	{
	    /**
	     * 返回请求过滤器
	     *
	     * @param \Illuminate\Http\Request $request
	     * @param \Closure $next
	     * @return mixed
	     */
	    public function handle($request, Closure $next)
	    {
	        if ($request->input('age') <= 200) {
	            return redirect('home');
	        }

	        return $next($request);
	    }

	}

## 执行前后的区别

	 public function handle($request, Closure $next)
	    {
	        // 执行动作

	        return $next($request);
	    }

	public function handle($request, Closure $next)
	    {
	        $response = $next($request);

	        // 执行动作

	        return $response;
	    }

## 注册

### 全局中间件

将相应的中间件类设置到 app/Http/Kernel.php 的数组属性 `$middleware` 中即可。

### 局部

首先：在 app/Http/Kernel.php 文件中分配给该中间件一个简写的 key。

// 在 App\Http\Kernel 里中
	protected $routeMiddleware = [
	    'auth' => \App\Http\Middleware\Authenticate::class,
	    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
	    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
	    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
	];


然后：指定

	Route::get('admin/profile', ['middleware' => 'auth', function () {
	    //
	}]);

	Route::get('/', ['middleware' => ['first', 'second'], function () {
	    //
	}]);

	Route::get('/', function () {
	    //
	})->middleware(['first', 'second']);

### 中间件组件

>通过指定一个键名的方式将相关中间件分到一个组里面，从而更方便将其分配到路由中，这可以通过使用 HTTP Kernel 的 $middlewareGroups  实现。

>中间件组的目的只是让一次分配给路由多个中间件的实现更加简单


	/**
	 * 应用的路由中间件组
	 *
	 * @var array
	 */
	protected $middlewareGroups = [
	    'web' => [
	        \App\Http\Middleware\EncryptCookies::class,
	        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
	        \Illuminate\Session\Middleware\StartSession::class,
	        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
	        \App\Http\Middleware\VerifyCsrfToken::class,
	    ],

	    'api' => [
	        'throttle:60,1',
	        'auth:api',
	    ],
	];

指定：

    Route::group(['middleware' => ['web']], function () {
	    //
	});

### 传参

	额外的中间件参数会在 $next 参数之后传入中间件：

	<?php

	namespace App\Http\Middleware;

	use Closure;

	class RoleMiddleware
	{
	    /**
	     * 运行请求过滤器
	     *
	     * @param \Illuminate\Http\Request $request
	     * @param \Closure $next
	     * @param string $role
	     * @return mixed
	     * translator http://laravelacademy.org
	     */
	    public function handle($request, Closure $next, $role)
	    {
	        if (! $request->user()->hasRole($role)) {
	            // Redirect...
	        }

	        return $next($request);
	    }

	}


	中间件参数可以在定义路由时通过：分隔中间件名和参数名来指定，多个中间件参数可以通过逗号分隔：

	Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
	    //
	}]);

### 可终止中间件









