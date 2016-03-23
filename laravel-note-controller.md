title: Laravel 笔记 - 控制器
date: 2016-03-23 22:12
tags: [Laravel]
categories: Laravel
---

MVC三巨头之一

<!-- more -->

---

>控制器可以将相关的 HTTP 请求封装到一个类中进行处理。通常控制器存放在 app/Http/Controllers 目录中。

# 书写

<?php

	namespace App\Http\Controllers;

	use App\User;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
	    /**
	     * 为指定用户显示详情
	     *
	     * @param int $id
	     * @return Response
	     */
	    public function showProfile($id)
	    {
	        return view('user.profile', ['user' => User::findOrFail($id)]);
	    }
	}


# 使用

    Route::get('user/{id}', 'UserController@showProfile');

>注：如果你在 App\Http\Controllers 目录下选择使用 PHP 命名空间嵌套或组织控制器，只需要使用相对于App\Http\Controllers 命名空间的指定类名即可。因此，如果你的完整控制器类是App\Http\Controllers\Photos\AdminController，你可以像这样注册路由：

	Route::get('foo', 'Photos\AdminController@method');

## 别名
	
	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

## 控制器中间件

	Route::get('profile', [
	    'middleware' => 'auth',
	    'uses' => 'UserController@showProfile'
	]);


	class UserController extends Controller
	{
	    /**
	     * 实例化一个新的 UserController 实例
	     *
	     * @return void
	     */
	    public function __construct()
	    {
	        $this->middleware('auth');
	        $this->middleware('log', ['only' => ['fooAction', 'barAction']]);
	        $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
	    }
	}

### RESTful 资源控制器

#### 创建

	//生成一个控制器文件 app/Http/Controllers/PhotoController.php
    php artisan make:controller PhotoController --resource
	
	//注册路由
	Route::resource('photo', 'PhotoController');

	//只定义部分资源路由
	Route::resource('photo', 'PhotoController',
	['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
	['except' => ['create', 'store', 'update', 'destroy']]);

	//覆盖这些默认的名字
	Route::resource('photo', 'PhotoController',
                ['names' => ['create' => 'photo.build']]);

## 依赖注入

### 构造函数注入


	<?php

	namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller
	{
	    /**
	     * The user repository instance.
	     */
	    protected $users;

	    /**
	     * 创建新的控制器实例
	     *
	     * @param UserRepository $users
	     * @return void
	     */
	    public function __construct(UserRepository $users)
	    {
	        $this->users = $users;
	    }
	}

### 方法注入

	//控制器方法期望输入路由参数，只需要将路由参数放到其他依赖之后
    Route::put('user/{id}', 'UserController@update');


	<?php

	namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller
	{
	    /**
	     * 存储新用户
	     *
	     * @param Request $request
	     * @return Response
	     */
	    public function store(Request $request, $id)
	    {
	        $name = $request->input('name');

	        //
	    }
	}



## 路由缓存

>如果你的应用完全基于控制器路由，可以使用 Laravel 的路由缓存。路由缓存不会作用于基于闭包的路由。要使用路由缓存，**必须**将闭包路由转化为控制器路由。


### 建立

    php artisan route:cache
    php artisan route:clear








