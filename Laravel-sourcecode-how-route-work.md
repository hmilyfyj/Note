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

# 开始之前

## 概念性

先了解路由的一些配置。

```php
Route::group(['as' => 'admin::','middleware' => 'auth', 'namespace' => 'AdminCon' , 'domain' => 'test.myapp.com', 'prefix' => 'Admin'], function () {
    Route::get('dashboard', ['as' => 'dashboard', function () {
    }]);
});
```
路由会产生这样的效果：
· 路由被**命名**为 "admin::dashboard",
- 送往路由前会经过 `auth` 中间件处理
- 匹配此路由的话，访问的控制器存放在 `App\Http\Controllers\AdminCon` 命名空间下。
- 匹配条件： `test.myapp.com/Admin/dashboard`

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

 `loadRoutes()` 调用了本类的 `map()` 函数。

```php
/**
     * Define the routes for the application.
     *
     * @param  \Illuminate\Routing\Router  $router
     * @return void
     */
    public function map(Router $router)
    {
        $router->group(['namespace' => $this->namespace], function ($router) {
            require app_path('Http/routes.php');
        });
    }

public function group(array $attributes, Closure $callback)
    {
        $this->updateGroupStack($attributes);

        // Once we have updated the group stack, we will execute the user Closure and
        // merge in the groups attributes when the route is created. After we have
        // run the callback, we will pop the attributes off of this group stack.
        call_user_func($callback, $this);

        array_pop($this->groupStack);
    }
```

`map()` 函数调用本类 `group()` 函数，看 `group()` 如何处理新注册的路由：

#### 第一步

`$this->updateGroupStack($attributes);` 这里主要是针对上一层 `group` 做的处理 `as` 、`prefix` 、`where` （暂时不知作用）、`as` 等参数，都需要和上一层 `group` 参数进行拼接。处理后入栈 `$this->groupStack` 。

#### 第二步

`call_user_func($callback, $this);` 调用传入的闭包，这里我们传入的是：

    require app_path('Http/routes.php');

即创建所有我们自定义的路由，拿一个举例：

	Route::get('/', function () { dump(['sss']); });

查看 `get()` 及其调用函数：

```php
public function get($uri, $action = null)
    {
        return $this->addRoute(['GET', 'HEAD'], $uri, $action);
    }

protected function addRoute($methods, $uri, $action)
    {
        return $this->routes->add($this->createRoute($methods, $uri, $action));
    }
```
简单讲，就是通过 `Route::get()` 方法创建了一个包含本次匹配规则的路由（Router） 对象，然后将其存入最开始定义好的 `$routes` 路由集合中。

看看 `createRoute` 如何创建`Route` 对象的：

```php
protected function createRoute($methods, $uri, $action)
    {
        // If the route is routing to a controller we will parse the route action into
        // an acceptable array format before registering it and creating this route
        // instance itself. We need to build the Closure that will call this out.
        if ($this->actionReferencesController($action)) {
            $action = $this->convertToControllerAction($action);
        }

        $route = $this->newRoute(
            $methods, $this->prefix($uri), $action
        );

        // If we have groups that need to be merged, we will merge them now after this
        // route has already been created and is ready to go. After we're done with
        // the merge we will be ready to return the route back out to the caller.
        if ($this->hasGroupStack()) {
            $this->mergeGroupAttributesIntoRoute($route);
        }

        $this->addWhereClausesToRoute($route);

        return $route;
    }
```

第一步就针对 `$action` 参数调用`Controller` 的处理，路由调用 `Controller` 有两种方式：

1. 字符串指定：Route::get('user/profile', 'UserController@showProfile');
2. 字符串指定放入数组内：Route::get('user/profile', [  'as' => 'profile', 'uses' =>  'UserController@showProfile' ]);

当满足以上两种情况之一时，`mergeGroupAttributesIntoRoute()` 返回 `true` ，调用 `convertToControllerAction()` 函数，不多说，跟进实现：

```php
protected function convertToControllerAction($action)
    {
        if (is_string($action)) {
            $action = ['uses' => $action];
        }

        // Here we'll merge any group "uses" statement if necessary so that the action
        // has the proper clause for this property. Then we can simply set the name
        // of the controller on the action and return the action array for usage.
        if (! empty($this->groupStack)) {
            $action['uses'] = $this->prependGroupUses($action['uses']);
        }

        // Here we will set this controller name on the action array just so we always
        // have a copy of it for reference if we need it. This can be used while we
        // search for a controller name or do some other type of fetch operation.
        $action['controller'] = $action['uses'];

        return $action;
    }
```

首先，取出字符串形式的 控制器动作，取出当前所在的`group` 配置，并将相应的命名空间拼接到控制器动作前。

然后，增加 `controller` 字段并传入处理过的控制器动作。最后就将处理过的`$action` 数组返回给 `createRoute()` 函数，继续看 `createRoute()` 的执行的命令：

	$route = $this->newRoute(
	            $methods, $this->prefix($uri), $action
	        );

```php
protected function newRoute($methods, $uri, $action)
    {
        return (new Route($methods, $uri, $action))
                    ->setRouter($this)
                    ->setContainer($this->container);
    }
```

利用处理好的参数，创建了一个`Route` 实例。下一步将执行：

	if ($this->hasGroupStack()) {
	            $this->mergeGroupAttributesIntoRoute($route);
	        }

如果该层路由存在于某个 `group` 下，则补充并合并 `$action` 参数（完整形式如下），然后设置其为`$route` 实例的action，最后将完全处理好可用的`$route` 对象返回，交给`addRoute()` 函数加入到`routes` 路由集当中，

```php
array:6 [▼
  "uses" => "App\Http\Controllers\UserController@test"
  "as" => "name"
  "controller" => "App\Http\Controllers\UserController@test"
  "namespace" => "App\Http\Controllers"
  "prefix" => null
  "where" => []
]
```

#### 第三步

本次 `group` 结束，出栈相关配置。

第二步最后一步里通过`add()` 方法将`$route` 放入结果集，结果集的参数有如下几种：

```php
//
$this->routes[$method][$domainAndUri] = $route;
$this->allRoutes[$method.$domainAndUri] = $route;

//lookup table
$this->nameList[$action['as']] = $route;
$this->actionList[trim($action['controller'], '\\')] = $route;
```

# 匹配阶段

`Kernel` 实例处理`$request` 的最后一程便是送往路由：

```php
return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());

protected function dispatchToRouter()
    {
        return function ($request) {
            $this->app->instance('request', $request);
            
            return $this->router->dispatch($request);
        };
    }
```

跟进看 `Router::dispatch()` 实现：

```php
public function dispatch(Request $request)
    {
        $this->currentRequest = $request;
        // 要详细看的 路由分发
        $response = $this->dispatchToRoute($request);
        
        return $this->prepareResponse($request, $response);
    }
```

跟进 `$response = $this->dispatchToRoute($request);` 的实现：

```php
public function dispatchToRoute(Request $request)
    {
        // First we will find a route that matches this request. We will also set the
        // route resolver on the request so middlewares assigned to the route will
        // receive access to this route instance for checking of the parameters.
        $route = $this->findRoute($request);
        
        $request->setRouteResolver(function () use ($route) {
            return $route;
        });

        $this->events->fire(new Events\RouteMatched($route, $request));

        $response = $this->runRouteWithinStack($route, $request);
        
        return $this->prepareResponse($request, $response);
    }
    
protected function findRoute($request)
    {
        $this->current = $route = $this->routes->match($request);
        
        $this->container->instance('Illuminate\Routing\Route', $route);
        
        return $this->substituteBindings($route);
    }
```

`dispatchToRoute()` 函数第一步就是调用 `findRoute()` 方法并通过 `$request` 对象匹配相应的路由， `findRoute()`  将调用路由容器实例 routes 的  `match` 方法匹配路由，跟进：

```php
public function match(Request $request)
    {
        $routes = $this->get($request->getMethod());
        
        // First, we will see if we can find a matching route for this current request
        // method. If we can, great, we can just return it so that it can be called
        // by the consumer. Otherwise we will check for routes with another verb.
        $route = $this->check($routes, $request);
        
        if (! is_null($route)) {
            return $route->bind($request);
        }

        // If no route was found we will now check if a matching route is specified by
        // another HTTP verb. If it is we will need to throw a MethodNotAllowed and
        // inform the user agent of which HTTP verb it should use for this route.
        $others = $this->checkForAlternateVerbs($request);
        
        if (count($others) > 0) {
            return $this->getRouteForMethods($request, $others);
        }

        throw new NotFoundHttpException;
    }
```
首先，通过 `请求方式`(如 `GET`、`POST` ) 取出路由数组。
然后，check 方法将对取出的路由数组与`$request` 请求一一对比

 `check()` 方法实现：

```php
protected function check(array $routes, $request, $includingMethod = true)
    {
        return Arr::first($routes, function ($key, $value) use ($request, $includingMethod) {
            return $value->matches($request, $includingMethod);
        });
    }
```


