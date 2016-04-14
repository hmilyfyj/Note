

title: Laravel 源码分析 -- 服务的注册、延迟启动
date: 2016-04-14 19:44
tags: [Laravel,PHP]
categories: Laravel
---

本文主要分析服务提供者如何启动、在哪里启动。

<!-- more -->

---

开始之前应当对 `ServiceProvider` 有一些基本的了解。

个人理解：`Laravel` 将根据不同的功能书写了不同的类库，这些类库统称为 服务（`Service`） ，为了解决不同类的加载、运行、延迟启动、以及依赖注入等麻烦事情，我们给每个`Service` 配置了一个“保姆“”，她们统称为`ServiceProvider` ，当`Application` 作为指挥者加载启动时，他可以按需调用`ServiceProvider` 的`register` 、`boot()` 等方法进行调度。

根据绑定时间的不同 `ServiceProvider` 可分为 3 种类型：`when`（事件触发型绑定） 、`eager` （立即绑定）、`deferred` (延迟绑定)。

上一篇笔记中说过，内核在将`$request` 请求交给中间件处理前，会对`$app` 进行一些处理，其中一步是调用`$this->bootstrap()` 函数，该函数将调用数个类的`bootstrap()` 方法，其中一个类叫做`Illuminate\Foundation\Bootstrap\RegisterProviders` ，用来注册需要的服务。

```php
/**
     * Register the application service providers.
     *
     * @param  array  $providers
     * @return void
     */
    public function load(array $providers)
    {
	    //第一步： 从缓存文件中读取配置清单
        $manifest = $this->loadManifest();
        
	    //第二步： 判断是否需要重新缓存编译配置 return is_null($manifest) || $manifest['providers'] != $providers;
        if ($this->shouldRecompile($manifest, $providers)) {
            $manifest = $this->compileManifest($providers);
        }
        
		//第三步：  绑定事件
        foreach ($manifest['when'] as $provider => $events) {
            $this->registerLoadEvents($provider, $events);
        }
        
        //第四步： 实例化并调用提供者的 register() 函数。
        foreach ($manifest['eager'] as $provider) {
            $this->app->register($this->createProvider($provider));
        }

        
        $this->app->addDeferredServices($manifest['deferred']);
    }
```
具体流程如注释。跟进每一步的代码来看。

第二步做了是否需要编译的判断，这里看看 `compileManifest()` 的实现：

```php
protected function compileManifest($providers)
    {
        // 修改成这种形式： ['providers' => $providers, 'eager' => [], 'deferred' => []];
        $manifest = $this->freshManifest($providers);
        
        // 遍历数组
        foreach ($providers as $provider) {
	        // 实例化：return new $provider($this->app);
            $instance = $this->createProvider($provider);

            // When recompiling the service manifest, we will spin through each of the
            // providers and check if it's a deferred provider or not. If so we'll
            // add it's provided services to the manifest and note the provider.
            // 需要延迟加载，如果是则存入 $manifest['deferred'][$service] 数组
            if ($instance->isDeferred()) {
                foreach ($instance->provides() as $service) {
                    $manifest['deferred'][$service] = $provider;
                }

				// 绑定事件
                $manifest['when'][$provider] = $instance->when();
            }

            // If the service providers are not deferred, we will simply add it to an
            // array of eagerly loaded providers that will get registered on every
            // request to this application instead of "lazy" loading every time.
            // 不需要延迟，加入 $manifest['eager'] 数组中。
            else {
                $manifest['eager'][] = $provider;
            }
        }
        
        // 写入缓存
        return $this->writeManifest($manifest);
    }
```

通过本函数处理，我们将得到这种结构的数组（做了些删减）：
	        
```php
// $manifest['deferred'][$service] = $provider;
// $manifest['when'][$provider] = $instance->when();
// $manifest['eager'][] = $provider;


array:4 [▼
  "when" => array:12 [▼
    "Illuminate\Broadcasting\BroadcastServiceProvider" => []
    "Illuminate\Bus\BusServiceProvider" => []
    "Illuminate\Cache\CacheServiceProvider" => []
  ]
  "providers" => array:2 [▼
    0 => "Illuminate\Auth\AuthServiceProvider"
    1 => "Illuminate\Broadcasting\BroadcastServiceProvider"
  ]
  "eager" => array:1 [▼
    0 => "Illuminate\Auth\AuthServiceProvider"
    1 => "Illuminate\Cookie\CookieServiceProvider"
  ]
  "deferred" => array:83 [▼
    "Illuminate\Broadcasting\BroadcastManager" => "Illuminate\Broadcasting\BroadcastServiceProvider"
    "Illuminate\Contracts\Broadcasting\Factory" => "Illuminate\Broadcasting\BroadcastServiceProvider"
  ]
]
```

这里先看 `registerLoadEvents()` 函数对 `when` 类型的服务提供者的处理如何实现：

	foreach ($manifest['when'] as $provider => $events) {
	            $this->registerLoadEvents($provider, $events);
	        }


```php
protected function registerLoadEvents($provider, array $events)
    {
        if (count($events) < 1) {
            return;
        }

        $app = $this->app;

        $app->make('events')->listen($events, function () use ($app, $provider) {
            $app->register($provider);
        });
    }
```

原来是实现了绑定`$event` 事件，当某事件触发时，将执行该`ServiceProvider` 的`register()` 函数。

再来看`$this->app->addDeferredServices($manifest['deferred']);` 对`deffered` 类型的处理：

首先要说明的是 `$services` 数组的构成，服务名为 `key`，该服务所依赖的服务提供者数组作为 `value`。

```php
public function addDeferredServices(array $services)
    {
        $this->deferredServices = array_merge($this->deferredServices, $services);
    }
```

将传入的`$services` 数组并入本对象的`deferredServices ` 数组中。那么问题来了，到底如何实现的延迟注册？还记得 `make()` 函数吗，当时只看了 Container 的 `make()` 实现，其实 `Application` 继承了`Container` 函数后对`make()` 方法进行了重写，就是为了处理延迟注册的问题，Have a look :

```php
public function make($abstract, array $parameters = [])
    {
        $abstract = $this->getAlias($abstract);

		// 检查 deferredServices 数组中是否存在依赖的需要延迟加载的服务。
        if (isset($this->deferredServices[$abstract])) {
            $this->loadDeferredProvider($abstract);
        }

        return parent::make($abstract, $parameters);
    }
```

在调用父类`Container` 的`make()` 函数进行实例化前，先检测有没有需要进行注册的`Service` 。

查看一下注册`loadDeferredProvider`的实现代码：


```php

public function loadDeferredProvider($service)
    {
	    // 没有依赖的服务
        if (! isset($this->deferredServices[$service])) {
            return;
        }
		
		// 获取依赖服务的服务提供者
        $provider = $this->deferredServices[$service];

		// 如果没被加载过。
        if (! isset($this->loadedProviders[$provider])) {
            $this->registerDeferredProvider($provider, $service);
        }
    }

public function registerDeferredProvider($provider, $service = null)
    {
        // Once the provider that provides the deferred service has been registered we
        // will remove it from our local list of the deferred services with related
        // providers so that this container does not try to resolve it out again.
        // 我们将服务依赖的服务注册后，依赖解决，就可以从数组中删除以这前者为key的元素了。
        if ($service) {
            unset($this->deferredServices[$service]);
        }

        $this->register($instance = new $provider($this));

		// 暂时不懂。略
        if (! $this->booted) {
            $this->booting(function () use ($instance) {
                $this->bootProvider($instance);
            });
        }
    }
```
