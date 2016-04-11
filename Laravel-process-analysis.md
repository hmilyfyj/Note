title: Laravel 流程分析
date: 2016-04-10 19:32
tags: [Laravel,PHP]
categories: Laravel
---

作为不读源码会死星人，不清楚实际运行流程会缺乏掌控感，我不喜欢。

<!-- more -->

---

本文初衷是想理一下`Laravel` 运作流程，在下一篇文章中将对各个阶段进行更加深入的了解。

# 从 index.php 看整体流程
 

```php
//一切都从public/index.php开始

//阶段一：加载 composer
require __DIR__ . '/../bootstrap/autoload.php';

//阶段二：实例化 Application 实例
$app = require_once __DIR__ . '/../bootstrap/app.php';

//阶段三：实例化 kernel 类，并把请求交给 $kernel->handle()
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
	$request = Illuminate\Http\Request::capture()
);

//阶段四：返回结果
$response->send();

$kernel->terminate($request, $response);
```
# 跟进各个阶段

## 阶段二：实例化 Application 实例

看`/../bootstrap/app.php`源码。

```php

$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);

//共享型  $this->bind($abstract, $concrete, true); 
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;
```

阶段二首先实例化了容器类：`Illuminate\Foundation\Application`，然后创建了单例模式的 Kernel 和 Exception 实例。我们跟进看一下`Illuminate\Foundation\Application` 的构造函数：

```php
public function __construct($basePath = null)
    {
	    //保存本实例。
        $this->registerBaseBindings();

		//注册基础的服务。
        $this->registerBaseServiceProviders();

		//注册核心容器的别名
        $this->registerCoreContainerAliases();

		//啥？
        if ($basePath) {
            $this->setBasePath($basePath);
        }
    }
```

一行行来跟进：

	   $this->registerBaseBindings(); 

```php
protected function registerBaseBindings()
    {
        static::setInstance($this);

        $this->instance('app', $this);

        $this->instance('Illuminate\Container\Container', $this);
    }
```
本函数将 application 实例分别保存到了：`static::$instance` 、`$this->instances['app']`、`$this->instances['Illuminate\Container\Container']`中。

	$this->registerBaseServiceProviders();
	
```php
protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }
```

本函数分别执行了两个服务提供者的 register 函数，稍后跟进。

	$this->registerCoreContainerAliases();

```php
public function registerCoreContainerAliases()
    {
        $aliases = [
            'app'                  => ['Illuminate\Foundation\Application', 'Illuminate\Contracts\Container\Container', 'Illuminate\Contracts\Foundation\Application'],
            'auth'                 => ['Illuminate\Auth\AuthManager', 'Illuminate\Contracts\Auth\Factory'],
            'auth.driver'          => ['Illuminate\Contracts\Auth\Guard'],
            'blade.compiler'       => ['Illuminate\View\Compilers\BladeCompiler'],
            'cache'                => ['Illuminate\Cache\CacheManager', 'Illuminate\Contracts\Cache\Factory'],
            'cache.store'          => ['Illuminate\Cache\Repository', 'Illuminate\Contracts\Cache\Repository'],
            'config'               => ['Illuminate\Config\Repository', 'Illuminate\Contracts\Config\Repository'],
            'cookie'               => ['Illuminate\Cookie\CookieJar', 'Illuminate\Contracts\Cookie\Factory', 'Illuminate\Contracts\Cookie\QueueingFactory'],
            'encrypter'            => ['Illuminate\Encryption\Encrypter', 'Illuminate\Contracts\Encryption\Encrypter'],
            'db'                   => ['Illuminate\Database\DatabaseManager'],
            'db.connection'        => ['Illuminate\Database\Connection', 'Illuminate\Database\ConnectionInterface'],
            'events'               => ['Illuminate\Events\Dispatcher', 'Illuminate\Contracts\Events\Dispatcher'],
            'files'                => ['Illuminate\Filesystem\Filesystem'],
            'filesystem'           => ['Illuminate\Filesystem\FilesystemManager', 'Illuminate\Contracts\Filesystem\Factory'],
            'filesystem.disk'      => ['Illuminate\Contracts\Filesystem\Filesystem'],
            'filesystem.cloud'     => ['Illuminate\Contracts\Filesystem\Cloud'],
            'hash'                 => ['Illuminate\Contracts\Hashing\Hasher'],
            'translator'           => ['Illuminate\Translation\Translator', 'Symfony\Component\Translation\TranslatorInterface'],
            'log'                  => ['Illuminate\Log\Writer', 'Illuminate\Contracts\Logging\Log', 'Psr\Log\LoggerInterface'],
            'mailer'               => ['Illuminate\Mail\Mailer', 'Illuminate\Contracts\Mail\Mailer', 'Illuminate\Contracts\Mail\MailQueue'],
            'auth.password'        => ['Illuminate\Auth\Passwords\PasswordBrokerManager', 'Illuminate\Contracts\Auth\PasswordBrokerFactory'],
            'auth.password.broker' => ['Illuminate\Auth\Passwords\PasswordBroker', 'Illuminate\Contracts\Auth\PasswordBroker'],
            'queue'                => ['Illuminate\Queue\QueueManager', 'Illuminate\Contracts\Queue\Factory', 'Illuminate\Contracts\Queue\Monitor'],
            'queue.connection'     => ['Illuminate\Contracts\Queue\Queue'],
            'redirect'             => ['Illuminate\Routing\Redirector'],
            'redis'                => ['Illuminate\Redis\Database', 'Illuminate\Contracts\Redis\Database'],
            'request'              => ['Illuminate\Http\Request', 'Symfony\Component\HttpFoundation\Request'],
            'router'               => ['Illuminate\Routing\Router', 'Illuminate\Contracts\Routing\Registrar'],
            'session'              => ['Illuminate\Session\SessionManager'],
            'session.store'        => ['Illuminate\Session\Store', 'Symfony\Component\HttpFoundation\Session\SessionInterface'],
            'url'                  => ['Illuminate\Routing\UrlGenerator', 'Illuminate\Contracts\Routing\UrlGenerator'],
            'validator'            => ['Illuminate\Validation\Factory', 'Illuminate\Contracts\Validation\Factory'],
            'view'                 => ['Illuminate\View\Factory', 'Illuminate\Contracts\View\Factory'],
        ];

        foreach ($aliases as $key => $aliases) {
            foreach ($aliases as $alias) {
                $this->alias($key, $alias);
            }
        }
    }
```
本函数将提前定义好的别名数组通过 `$this->alias($key, $alias);` 方式保存到容器中，即：`$this->aliases[$alias] = $this->normalize($abstract);`。


## 阶段三：实例化 kernel 类，并把请求交给 $kernel->handle()

	
	$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

	$response = $kernel->handle(
		$request = Illuminate\Http\Request::capture()
	);

`$kernel`为阶段二中通过 app->singleton 函数创建的`App\Http\Kernel::class` 类实例。

当我们通过HTTP方式访问时，$kernel 实例是通过`App\Http\Kernel` 类实现的，这个类继承了`Illuminate\Foundation\Http\Kernel`。 request 请求将交给 kernel 实例的`handle()`方法处理。

```php
 /**
     * Handle an incoming HTTP request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();

            $response = $this->sendRequestThroughRouter($request);
        } catch (Exception $e) {
            $this->reportException($e);

            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));

            $response = $this->renderException($request, $e);
        }

        $this->app['events']->fire('kernel.handled', [$request, $response]);

        return $response;
    }
```

跟进 `$this->sendRequestThroughRouter($request);` 方法：

```php
protected function sendRequestThroughRouter($request)
    {
    //保存 $request 实例。
        $this->app->instance('request', $request);
        
        //unset(static::$resolvedInstance[$name]);
        Facade::clearResolvedInstance('request');

		//稍后跟进。
        $this->bootstrap();

		//流水线
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```

接着看`$this->bootstrap();` 代码：

```php
/**
     * Bootstrap the application for HTTP requests.
     *
     * @return void
     */
    public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }


/**
     * Get the bootstrap classes for the application.
     *
     * @return array
     */
    protected function bootstrappers()
    {
        return $this->bootstrappers;
    }

/**
     * The bootstrap classes for the application.
     *
     * @var array
     */
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
`bootstrap()` 函数调用了 `$app` 实例的`bootstartpWith()` 方法，并将本类的`$this->bootstrappers` 数组作为参数传入，继续查看 `bootstartpWith()` 方法： 

```php
/**
     * Run the given array of bootstrap classes.
     *
     * @param  array  $bootstrappers
     * @return void
     */
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

本函数将迭代实例化传入数组中的每一个类并调用她们的 bootstrap() 方法。那些类？ 阶段三最开始提到的：

	/**
	     * The bootstrap classes for the application.
	     *
	     * @var array
	     */
	    protected $bootstrappers = [
	        'Illuminate\Foundation\Bootstrap\DetectEnvironment',
	        'Illuminate\Foundation\Bootstrap\LoadConfiguration',
	        'Illuminate\Foundation\Bootstrap\ConfigureLogging',
	        'Illuminate\Foundation\Bootstrap\HandleExceptions',
	        'Illuminate\Foundation\Bootstrap\RegisterFacades',
	        'Illuminate\Foundation\Bootstrap\RegisterProviders',
	        'Illuminate\Foundation\Bootstrap\BootProviders',
	    ];

根据名称我们可以看到`bootstrap()` 函数的运行流程： 检测环境=》导入配置=》配置日志=》处理异常=》注册门面类=》注册服务=》启动服务。这些东西下一次详细说。

至此，程序已经运转起来，也就是我们的各种设备已经准备就绪，接下来就要把货`$request` 送进流水线进行加工处理了。也就是一开始看到的：

	(new Pipeline($this->app))
	                    ->send($request)
	                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
	                    ->then($this->dispatchToRouter());

流水线的尽头是 Router，我们将最终得到的 `$response` 实例返回，调用其 `send()` 方法，返回给请求者。

