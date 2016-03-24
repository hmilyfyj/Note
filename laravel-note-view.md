
title: Laravel 笔记 - 视图
date: 2016-03-24 09:58
tags: [Laravel]
categories: Laravel
---

RT

<!-- more -->

---

视图文件存放在 resources/views 目录。

## 简单使用

	Route::get('/', function () {
	    return view('greeting', ['name' => 'James']);
	});

	//嵌套：视图存放路径是resources/views/admin/profile.php
	return view('admin.profile', $data);

	//判断是否存在
	if (view()->exists('emails.customer')) {
	    //
	}

	$view = view('greeting')->with('name', 'Victoria');

	//共享数据片段:修改提供者如AppServiceProvider的boot()函数 
	<?php

	namespace App\Providers;

	class AppServiceProvider extends ServiceProvider
	{
	    /**
	     * 启动所有应用服务
	     *
	     * @return void
	     */
	    public function boot()
	    {
	        view()->share('key', 'value');
	    }

	    /**
	     * 注册服务提供者
	     *
	     * @return void
	     */
	    public function register()
	    {
	        //
	    }
	}


## 视图 Composer

autoload

	<?php

	namespace App\Providers;

	use Illuminate\Support\ServiceProvider;

	class ComposerServiceProvider extends ServiceProvider
	{
	    /**
	     * 在容器中注册绑定.
	     *
	     * @return void
	     * @author http://laravelacademy.org
	     */
	    public function boot()
	    {
	        // 使用基于类的composers...
	        view()->composer(
	            'profile', 'App\Http\ViewComposers\ProfileComposer'
	        );

			// 为多个视图绑定
			view()->composer(
			    ['profile', 'dashboard'],
			    'App\Http\ViewComposers\
			);

			view()->composer('*', function ($view) {
			    //
			});



	        // 使用基于闭包的composers...
	        view()->composer('dashboard', function ($view) {
	        });
	    }

	    /**
	     * 注册服务提供者.
	     *
	     * @return void
	     */
	    public function register()
	    {
	        //
	    }
	}

每次 profile 视图被渲染时都会执行 ProfileComposer@compose


	<?php

	namespace App\Http\ViewComposers;

	use Illuminate\Contracts\View\View;
	use Illuminate\Users\Repository as UserRepository;

	class ProfileComposer
	{
	    /**
	     * 用户仓库实现.
	     *
	     * @var UserRepository
	     */
	    protected $users;

	    /**
	     * 创建一个新的属性composer.
	     *
	     * @param UserRepository $users
	     * @return void
	     */
	    public function __construct(UserRepository $users)
	    {
	        // Dependencies automatically resolved by service container...
	        $this->users = $users;
	    }

	    /**
	     * 绑定数据到视图.
	     *
	     * @param View $view
	     * @return void
	     */
	    public function compose(View $view)
	    {
	        $view->with('count', $this->users->count());
	    }
	}

## 视图创建器

## Blade 模版

Blade 视图文件使用 .blade.php 文件扩展并存放在 resources/views 目录下。


### 模版继承

#### 举例

	<!-- 存放在 resources/views/layouts/master.blade.php -->

	<html>
	    <head>
	        <title>App Name - @yield('title')</title>
	    </head>
	    <body>
	        @section('sidebar')
	            This is the master sidebar.
	        @show

	        <div class="container">
	            @yield('content')
	        </div>
	    </body>
	</html>


	<!-- 存放在 resources/views/layouts/child.blade.php -->

	@extends('layouts.master')

	@section('title', 'Page Title')

	@section('sidebar')
	    @parent

	    <p>This is appended to the master sidebar.</p>
	@endsection

	@section('content')
	    <p>This is my body content.</p>
	@endsection

>sidebar 片段使用 @parent 指令来追加（而非覆盖）内容到布局中 sidebar，@parent 指令在视图渲染时将会被布局中的内容替换。


### 数据显示

	Hello, \{\{ $name }}.

	The current UNIX timestamp is \{\{ time() }}.

	//不渲染
	Hello, @\{\{ name }}.

	//改进三元运算
	\{\{ $name or 'Default' }}

	//避免被处理
	Hello, {!! $name !!}.

>注：已经经过 PHP 的 htmlentities 函数处理


### 流程控制

#### IF

	@if (count($records) === 1)
	    I have one record!
	@elseif (count($records) > 1)
	    I have multiple records!
	@else
	    I don't have any records!
	@endif

	@unless (Auth::check())
	    You are not signed in.
	@endunless

#### 循环

	@for ($i = 0; $i < 10; $i++)
	    The current value is \{\{ $i }}
	@endfor

	@foreach ($users as $user)
	    <p>This is user \{\{ $user->id }}</p>
	@endforeach

	@forelse ($users as $user)
	    <li>\{\{ $user->name }}</li>
	    @empty
	    <p>No users</p>
	@endforelse

	@while (true)
	    <p>I'm looping forever.</p>
	@endwhile

### 包含子视图
	
	<div>
	    @include('shared.errors')

	    <form>
	        <!-- Form Contents -->
	    </form>
	</div>

	//传递额外参数到被包含的视图：
	@include('view.name', ['some' => 'data'])

	//注释 不会包含在html中
	\{\{-- This comment will not be present in the rendered HTML --}}


>注：不要在 Blade 视图中使用 __DIR__ 和 __FILE__ 常量，因为它们会指向缓存视图的路径。

## 服务注入

@inject 指令可以用于从服务容器中获取服务，传递给 @inject 的第一个参数是服务将要被放置到的变量名，第二个参数是要解析的服务类名或接口名：

	@inject('metrics', 'App\Services\MetricsService')

	<div>
	    Monthly Revenue: \{\{ $metrics->monthlyRevenue() }}.
	</div>



## 扩展Blade

	//创建了一个 @datetime($var) 指令格式化给定的 $var：
	<?php

	namespace App\Providers;

	use Blade;
	use Illuminate\Support\ServiceProvider;

	class AppServiceProvider extends ServiceProvider
	{
	    /**
	     * Perform post-registration booting of services.
	     *
	     * @return void
	     */
	    public function boot()
	    {
	        Blade::directive('datetime', function($expression) {
	            return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
	        });
	    }

	    /**
	     * 在容器中注册绑定.
	     *
	     * @return void
	     */
	    public function register()
	    {
	        //
	    }
	}






