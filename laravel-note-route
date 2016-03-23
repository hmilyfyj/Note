title: Laravel 笔记 - 路由
date: 2016-03-23 19:38
tags: [Laravel]
categories: Laravel
---

RT

<!-- more -->

---

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

### 显式绑定


显式绑定，需要使用路由的 model 方法来为给定参数指定绑定类。应该在 RouteServiceProvider::boot 方法中定义模型绑定：

绑定参数到模型

	public function boot(Router $router)
	{
	    parent::boot($router);
	    $router->model('user', 'App\User');
	}

自定义解析

	$router->bind('user', function($value) {
	    return App\User::where('name', $value)->first();
	});

自定义“Not Found”

如果你想要指定自己的“Not Found”行为，将封装该行为的闭包作为第三个参数传递给 model 方法：

	$router->model('user', 'App\User', function() {
	    throw new NotFoundHttpException;
	});

### 伪造表单

HTML 表单不支持 PUT、PATCH 或者 DELETE 请求方法，因此，当 PUT、PATCH 或 DELETE 路由时，需要添加一个隐藏的 _method 字段到表单中，其值被用作该表单的 HTTP 请求方法：


	<form action="/foo/bar" method="POST">
	    <input type="hidden" name="_method" value="PUT">
	    <input type="hidden" name="_token" value="{{ csrf_token() }}">
	</form>

还可以使用辅助函数 method_field 来实现这一目的：

	<?php echo method_field('PUT'); ?>

当然，也支持 Blade 模板引擎：

	{{ method_field('PUT') }}

