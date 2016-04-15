title: Laravel 源码分析 -- Route 路由可以这么玩。
date: 2016-04-15 22:55
tags: [Laravel,PHP]
categories: Laravel
---

顺着路由的走向，从启动到分发，一步步走，一步两步，一步两步，一步一步似爪牙，似魔鬼的步伐...

`handle()` 处理`$request` 时，流水线的最后一棒，就是交由路由分发：

```php
return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
```

今天就看看，路由是怎么配置、匹配、生效的。

<!-- more -->

---

# 配置阶段

## 注册服务

在`Application` 启动初期，就注册了 `Route` 服务：

	protected function registerBaseServiceProviders()
	    {
	        $this->register(new EventServiceProvider($this));

	        $this->register(new RoutingServiceProvider($this));
	    }


`RoutingServiceProvider`  的 `register()` 函数做了这样的事情：

```php
public function register()
    {
        $this->registerRouter();

        $this->registerUrlGenerator();

        $this->registerRedirector();

        $this->registerPsrRequest();

        $this->registerPsrResponse();

        $this->registerResponseFactory();
    }
```

`register()` 方法第一步就是注册 `Router` 类。 

```php
$this->app['router'] = $this->app->share(function ($app) {
            return new Router($app['events'], $app);
        });
```

看一下路由的构造函数：

```php
public function __construct(Dispatcher $events, Container $container = null)
    {
        // 事件对象。
        $this->events = $events;
        
        // routes 集合。
        $this->routes = new RouteCollection;
        
        // Application 实例。
        $this->container = $container ?: new Container;

        // 暂时不懂，略过。
        $this->bind('_missing', function ($v) {
            return explode('/', $v);
        });
    }
```

## 注册路由

提到多次的`Kernel::bootstrap()` 函数启动了`RouteServiceProvider` 类实例的 `boot()` 函数。

看实现：

```php
public function boot(Router $router)
    {
        $this->setRootControllerNamespace();

        if ($this->app->routesAreCached()) {
            $this->loadCachedRoutes();
        } else {
            $this->loadRoutes();
            
            $this->app->booted(function () use ($router) {
                $router->getRoutes()->refreshNameLookups();
            });
        }
    }
```

### 已缓存

			$this->loadCachedRoutes();

### 未缓存

			$this->loadRoutes();
            
            $this->app->booted(function () use ($router) {
                $router->getRoutes()->refreshNameLookups();
            });



# 匹配阶段






