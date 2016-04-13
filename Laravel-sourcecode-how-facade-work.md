
title: Laravel 源码分析 -- Facade 原来是这么玩的
date: 2016-04-13 10:02
tags: [Laravel,PHP]
categories: Laravel
---

Route如何用静态方法来处理路由的？ 因为`Facade`。

<!-- more -->

---

# 流程

## 从Kernel 开始

```php
protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);
        
        Facade::clearResolvedInstance('request');

        $this->bootstrap();
        
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```

在将请求送往路由器之前，需要做一些几本功能的启动：

    $this->bootstrap();



# 流程

## 从Kernel 开始

```php
protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);
        
        Facade::clearResolvedInstance('request');

        $this->bootstrap();
        
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```

在将请求送往路由器之前，需要做一些几本功能的启动：

    $this->bootstrap();

它做了什么？ 看一下：

```php
public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }

protected $bootstrappers = [
        'Illuminate\Foundation\Bootstrap\DetectEnvironment',
        'Illuminate\Foundation\Bootstrap\LoadConfiguration',
        'Illuminate\Foundation\Bootstrap\ConfigureLogging',
        'Illuminate\Foundation\Bootstrap\HandleExceptions',
        'Illuminate\Foundation\Bootstrap\RegisterFacades',
        'Illuminate\Foundation\Bootstrap\RegisterProviders',
        'Illuminate\Foundation\Bootstrap\BootProviders',
    ];
```

看看`Application` 下的`enter code here` 方法：

```php
public function bootstrapWith(array $bootstrappers)
    {
        $this->hasBeenBootstrapped = true;
        
        foreach ($bootstrappers as $bootstrapper) {
            $this['events']->fire('bootstrapping: '.$bootstrapper, [$this]);

            $this->make($bootstrapper)->bootstrap($this);

            $this['events']->fire('bootstrapped: '.$bootstrapper, [$this]);
        }
    }
```


原来是迭代调用`Kernel` 类内`$bootstrappers` 数组中类的`bootstrap($this)` 方法。

在数组中我们看到一个：

    'Illuminate\Foundation\Bootstrap\RegisterFacades'

这就是我们今天要研究的 `Facade` 入口。

## RegisterFacades

`RegisterFacades` 类只有一个方法：

```php
public function bootstrap(Application $app)
    {
	    //清除已解析的实例 static::$resolvedInstance = [];
        Facade::clearResolvedInstances();
		//设置 $app 实例。static::$app = $app;
        Facade::setFacadeApplication($app);
        
        //注册
        AliasLoader::getInstance($app->make('config')->get('app.aliases'))->register();
    }
```

前两句比较简单，做了下环境的初始化，直接看第三句：

    AliasLoader::getInstance($app->make('config')->get('app.aliases'))->register();
    

首先，获取aliases配置

    $app->make('config')->get('app.aliases')

执行后，我们将得到在 cofnig/app.php 内定义的 aliases 数组：

```php
'aliases' => [

		'App' => Illuminate\Support\Facades\App::class,
		'Artisan' => Illuminate\Support\Facades\Artisan::class,
		'Auth' => Illuminate\Support\Facades\Auth::class,
		'Blade' => Illuminate\Support\Facades\Blade::class,
		'Cache' => Illuminate\Support\Facades\Cache::class,
		'Config' => Illuminate\Support\Facades\Config::class,
		'Cookie' => Illuminate\Support\Facades\Cookie::class,
		'Crypt' => Illuminate\Support\Facades\Crypt::class,
		'DB' => Illuminate\Support\Facades\DB::class,
		'Eloquent' => Illuminate\Database\Eloquent\Model::class,
		'Event' => Illuminate\Support\Facades\Event::class,
		'File' => Illuminate\Support\Facades\File::class,
		'Gate' => Illuminate\Support\Facades\Gate::class,
		'Hash' => Illuminate\Support\Facades\Hash::class,
		'Lang' => Illuminate\Support\Facades\Lang::class,
		'Log' => Illuminate\Support\Facades\Log::class,
		'Mail' => Illuminate\Support\Facades\Mail::class,
		'Password' => Illuminate\Support\Facades\Password::class,
		'Queue' => Illuminate\Support\Facades\Queue::class,
		'Redirect' => Illuminate\Support\Facades\Redirect::class,
		'Redis' => Illuminate\Support\Facades\Redis::class,
		'Request' => Illuminate\Support\Facades\Request::class,
		'Response' => Illuminate\Support\Facades\Response::class,
		'Route' => Illuminate\Support\Facades\Route::class,
		'Schema' => Illuminate\Support\Facades\Schema::class,
		'Session' => Illuminate\Support\Facades\Session::class,
		'Storage' => Illuminate\Support\Facades\Storage::class,
		'URL' => Illuminate\Support\Facades\URL::class,
		'Validator' => Illuminate\Support\Facades\Validator::class,
		'View' => Illuminate\Support\Facades\View::class,

	]
```

然后，别名数组作为参数传入` AliasLoader::getInstance()` 中，顾名思义，获取`AliasLoader` 的实例：

```php
public static function getInstance(array $aliases = [])
    {
        if (is_null(static::$instance)) {
            return static::$instance = new static($aliases);
        }
        
        $aliases = array_merge(static::$instance->getAliases(), $aliases);

        static::$instance->setAliases($aliases);

        return static::$instance;
    }
```

将传入的数组并入本类的静态参数`$this->aliases` 内，并根据是否被实例化过做了不同的处理，不必细说。

最后，调用了`register()` 方法，这里是有趣的开始：

    AliasLoader::getInstance($app->make('config')->get('app.aliases'))->register();

`register()` 实现：

```php
public function register()
    {
        if (! $this->registered) {
            $this->prependToLoaderStack();

			//标记为已注册过。
            $this->registered = true;
        }
    }
```

未注册过时，将调用`prependToLoaderStack()` 函数，它做了什么？看源码：

```php
protected function prependToLoaderStack()
    {
        spl_autoload_register([$this, 'load'], true, true);
    }
```

原来是将本类`load` 函数作为`__autoload` 的实现，并且处于解析队列之首。当我们无法在某个命名空间下找到类时，将首先调用`AliasLoader` 实例的 `load()` 方法。

接着看 `load()` 的实现：

```php
public function load($alias)
    {
        if (isset($this->aliases[$alias])) {
            return class_alias($this->aliases[$alias], $alias);
        }
    }
``` 

很简单，会通过 `class_alias` 方法定义别名并返回处理结果，此时别名和原有类将完全相同。


此时，我已经可以愉快的调用 Router 而不抱错，但是，如果我想调用 Router::get，仅仅这样是不行的， `Illuminate\Support\Facades\Route` 本身并不带任何路由处理的方法的实现，甚至根本找不到`get()` 方法，这里说到了找不到，类找不到可以调用`__autoload` ，静态方法找不到，同样可以调用`__callStatic` 方法，看一下源码：

```php
public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        switch (count($args)) {
            case 0:
                return $instance->$method();

            case 1:
                return $instance->$method($args[0]);

            case 2:
                return $instance->$method($args[0], $args[1]);

            case 3:
                return $instance->$method($args[0], $args[1], $args[2]);

            case 4:
                return $instance->$method($args[0], $args[1], $args[2], $args[3]);

            default:
                return call_user_func_array([$instance, $method], $args);
        }
    }
```

原来是获取了实例后调用的`get` 方法，离真相越来越近了。看一下获取实例的方法`static::getFacadeRoot()` 的实现：

该实现在`Route` 的父类`Facade` 中：

```php
public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }

protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        return static::$resolvedInstance[$name] = static::$app[$name];
    }
```

`getFacadeAccessor`方法在`Route` 类中被重写：

```php
protected static function getFacadeAccessor()
    {
        return 'router';
    }
```


以 `Route` 为例，在调用`getFacadeRoot()` 方法后，会调用`Route` 的`static::getFacadeAccessor()` 方法，得到`router` 传入`resolveFacadeInstance()` 方法中，如果解析过、或者已经是对象，则直接返回对象，如果不是，则调用`Application` 实例的`make` 方法进行实例化，即`static::$app[$name]` ，而`$app['router']` 恰恰是我们之前已经注册过的路由。

 
```php
protected function registerRouter()
    {
        $this->app['router'] = $this->app->share(function ($app) {
            return new Router($app['events'], $app);
        });
    }

```

