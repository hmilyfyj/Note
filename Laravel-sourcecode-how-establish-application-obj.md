title: Laravel 源码分析 -- Application 的搭建
date: 2016-04-12 15:02
tags: [Laravel,PHP]
categories: Laravel
---

之前粗略的分析了一次HTTP请求需要经过的步骤，现在来具体看其中的一步：Application 实例的创建。

<!-- more -->

---

```php
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);
```

一起来看下 Application 类的声明及它的构造函数：

```php
namespace Illuminate\Foundation;

use Closure;
use RuntimeException;
use Illuminate\Support\Arr;
use Illuminate\Support\Str;
use Illuminate\Http\Request;
use Illuminate\Container\Container;
use Illuminate\Filesystem\Filesystem;
use Illuminate\Support\ServiceProvider;
use Illuminate\Events\EventServiceProvider;
use Illuminate\Routing\RoutingServiceProvider;
use Symfony\Component\HttpKernel\HttpKernelInterface;
use Symfony\Component\HttpKernel\Exception\HttpException;
use Symfony\Component\HttpFoundation\Request as SymfonyRequest;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Illuminate\Contracts\Foundation\Application as ApplicationContract;

class Application extends Container implements ApplicationContract, HttpKernelInterface
{

public function __construct($basePath = null)
    {
	    //保存本实例。
        $this->registerBaseBindings();

		//注册基础的服务。
        $this->registerBaseServiceProviders();

		//注册核心容器的别名
        $this->registerCoreContainerAliases();

        if ($basePath) {
            $this->setBasePath($basePath);
        }
    }
}
```

# 1 Step

我们来看第一句`$this->registerBaseBindings();` 实现：

```php
/**
     * Register the basic bindings into the container.
     *
     * @return void
     */
    protected function registerBaseBindings()
    {
        static::setInstance($this);

        $this->instance('app', $this);

        $this->instance('Illuminate\Container\Container', $this);
    }
```

这个函数上一篇文章中说过

>本函数将 application 实例分别保存到了：`static::$instance` 、`$this->instances['app']`、`$this->instances['Illuminate\Container\Container']`中。

看一下 `$this->instance('app',$this)` 的实现：

```php
public function instance($abstract, $instance)
    {
        $abstract = $this->normalize($abstract);

        // 首先判断$abstract抽象体是否是个数组，如果是，我们就将数组
        // 拆成键值对拿到$abstract和它的别名，并将它们通过alias函数
        //将它们以aliases[$alias] = $abstract的形式放到aliases数组
        // 里面方便以后查找，然后删除alias数组中以$abstract为键的项以免接下来的绑定判断出现问题
        if (is_array($abstract)) {
            list($abstract, $alias) = $this->extractAlias($abstract);

            $this->alias($abstract, $alias);
        }

        unset($this->aliases[$abstract]);

        // 检测是否绑定过 
        $bound = $this->bound($abstract);

		//放入 instances[] 数组中。
        $this->instances[$abstract] = $instance;

		//重新绑定
        if ($bound) {
            $this->rebound($abstract);
        }
    }

 public function bound($abstract)
    {
        return isset($this->bindings[$abstract]) || isset($this->instances[$abstract]) || $this->isAlias($abstract);
    }
```

# 2 Step

	$this->registerBaseServiceProviders();

```php
protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }
```

得到服务实例后传入 register() 函数，看看它做了什么：

```php
public function register($provider, $options = [], $force = false)
    {
        if (($registered = $this->getProvider($provider)) && ! $force) {
            return $registered;
        }

        // If the given "provider" is a string, we will resolve it, passing in the
        // application instance automatically for the developer. This is simply
        // a more convenient way of specifying your service provider classes.
        if (is_string($provider)) {
            $provider = $this->resolveProviderClass($provider);
        }

        $provider->register();

        // Once we have registered the service we will iterate through the options
        // and set each of them on the application so they will be available on
        // the actual loading of the service objects and for developer usage.
        foreach ($options as $key => $value) {
            $this[$key] = $value;
        }

        $this->markAsRegistered($provider);

        // If the application has already booted, we will call this boot method on
        // the provider class so it has an opportunity to do its boot logic and
        // will be ready for any usage by the developer's application logics.
        if ($this->booted) {
            $this->bootProvider($provider);
        }

        return $provider;
    }
```

### register 流程

#### 1.检测是否注册过

如果注册过就直接返回。

	if (($registered = $this->getProvider($provider)) && ! $force) {
	            return $registered;
	        }
看看 `getProvider()` 的实现。

```php
public function getProvider($provider)
    {
        $name = is_string($provider) ? $provider : get_class($provider);

        return Arr::first($this->serviceProviders, function ($key, $value) use ($name) {
            return $value instanceof $name;
        });
    }
```

首先判断了传入的`provider`类型，如果是类则取其类名。然后通过自建的 `Arr::first` 方法查找相符的 provider。

Array::first() 的实现：

```php
public static function first($array, callable $callback = null, $default = null)
    {
        if (is_null($callback)) {
            return empty($array) ? value($default) : reset($array);
        }

        foreach ($array as $key => $value) {
            if (call_user_func($callback, $key, $value)) {
                return $value;
            }
        }

        return value($default);
    }
```

`first` 将迭代执行传入的闭包函数，找出`serviceProviders` 数组 中作为传入`$name` 类实例的`$value` 并实例化。

回到 `register()` 函数，执行到第二步

	if (is_string($provider)) {
	            $provider = $this->resolveProviderClass($provider);
	        } 

```php
if (is_string($provider)) {
            $provider = $this->resolveProviderClass($provider);
        }

```


第三步，调用了传入 ServiceProvider 的`register` 方法。

第四步，将需要的参数注入当前 `application` 中。

```php
foreach ($options as $key => $value) {
            $this[$key] = $value;
        }
```

第五步，`$this->markAsRegistered($provider);` 标记该服务为已注册。

```php
protected function markAsRegistered($provider)
    {
        $this['events']->fire($class = get_class($provider), [$provider]);

        $this->serviceProviders[] = $provider;

        $this->loadedProviders[$class] = true;
    }
```

这里有个有意思的地方，它是通过`$this['events']` 方式调用 events 的，它是这样实现的，`Application` 的父类继承了`ArrayAccess` ，然后在内部实现了

```php
/**
     * Get the value at a given offset.
     *
     * @param  string  $key
     * @return mixed
     */
    public function offsetGet($key)
    {
        return $this->make($key);
    }
```

