title: Laravel 流程分析
date: 2016-04-10 19:32
tags: [Laravel,PHP]
categories: Laravel
---

我是不读源码会死星人，不清楚实际运行流程会别扭，缺乏掌控感。

<!-- more -->

---

# 从 index.php 看整体流程
 

```php
//一切都从public/index.php开始

//阶段一：加载 composer
require __DIR__ . '/../bootstrap/autoload.php';

//阶段二：实例化 Application 实例
$app = require_once __DIR__ . '/../bootstrap/app.php';

//阶段三：实例化 kernel 类，处理请求
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

阶段二实例化了容器类：`Illuminate\Foundation\Application`，并创建了单例模式的 Kernel 和 Exception 实例。继续看`Illuminate\Foundation\Application` 构造函数：

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



