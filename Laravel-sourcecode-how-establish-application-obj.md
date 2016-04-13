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

# 构造函数

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

## 1 Step

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

## 2 Step

	$this->registerBaseServiceProviders();

```php
protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }
```



得到服务实例后将其传入 register() 函数，看看它做了什么：

#### register() 函数

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
        // 调用其 boot 方法。
        if ($this->booted) {
            $this->bootProvider($provider);
        }

        return $provider;
    }
```



##### 1.检测是否注册过



	if (($registered = $this->getProvider($provider)) && ! $force) {
	            return $registered;
	        }
	        
如果注册过就直接返回。看看 `getProvider()` 的实现：

```php
public function getProvider($provider)
    {
        $name = is_string($provider) ? $provider : get_class($provider);

        return Arr::first($this->serviceProviders, function ($key, $value) use ($name) {
            return $value instanceof $name;
        });
    }
```

首先判断了传入的`provider`类型，如果是类则取其类名。然后通过自建的 `Arr::first` 方法在 `$this->serviceProviders` 数组中查找符合条件的 provider。

`Array::first()` 的实现：

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

`first` 将迭代执行传入的闭包函数，找出`serviceProviders` 数组 中`$name` 类的实例 -- `$value` 并实例化。

##### 2 检测是否为字符串

回到 `register()` 函数，执行到第二步

	if (is_string($provider)) {
	            $provider = $this->resolveProviderClass($provider);
	        } 

```php
public function resolveProviderClass($provider)
    {
        return new $provider($this);
    }
```


##### 3 调用服务提供者 register() 方法

    $provider->register();

##### 4 注入所需参数

将需要的参数注入当前 `application` 中。

```php
foreach ($options as $key => $value) {
            $this[$key] = $value;
        }
```

##### 5 标记该服务为已注册

    $this->markAsRegistered($provider);` 

`markAsRegistered()`函数的实现：

```php
protected function markAsRegistered($provider)
    {
        $this['events']->fire($class = get_class($provider), [$provider]);

        $this->serviceProviders[] = $provider;

        $this->loadedProviders[$class] = true;
    }
```

这里有个有意思的地方，第一句话，它是通过`$this['events']` 方式调用 events 。

它是这样实现的，`Application` 的父类继承了`ArrayAccess` ，然后在内部实现了

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

由于我们之前注册了`events` 服务：
```php
$this->app->singleton('events', function ($app) {
            return (new Dispatcher($app))->setQueueResolver(function () use ($app) {
                return $app->make('Illuminate\Contracts\Queue\Factory');
            });
        });
```

现在要调用`make` 函数，先看一下如何make的实现的吧：

###### \$app->make() 函数

```php
public function make($abstract, array $parameters = [])
    {
        $abstract = $this->getAlias($this->normalize($abstract));

        // If an instance of the type is currently being managed as a singleton we'll
        // just return an existing instance instead of instantiating new instances
        // so the developer can keep using the same objects instance every time.
        if (isset($this->instances[$abstract])) {
            return $this->instances[$abstract];
        }

        $concrete = $this->getConcrete($abstract);
        
        // We're ready to instantiate an instance of the concrete type registered for
        // the binding. This will instantiate the types, as well as resolve any of
        // its "nested" dependencies recursively until all have gotten resolved.
        if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete, $parameters);
        } else {
        	echo '1';
        	print_r($abstract);die('1');
            $object = $this->make($concrete, $parameters);
        }

        // If we defined any extenders for this type, we'll need to spin through them
        // and apply them to the object being built. This allows for the extension
        // of services, such as changing configuration or decorating the object.
        foreach ($this->getExtenders($abstract) as $extender) {
            $object = $extender($object, $this);
        }

        // If the requested type is registered as a singleton we'll want to cache off
        // the instances in "memory" so we can return it later without creating an
        // entirely new instance of an object on each subsequent request for it.
        if ($this->isShared($abstract)) {
            $this->instances[$abstract] = $object;
        }

        $this->fireResolvingCallbacks($abstract, $object);

        $this->resolved[$abstract] = true;

        return $object;
    }
```

按照流程捋一下make函数：

·1 `$abstract` 如果有别名则改为别名，是否已经实例化过。如果存在与isntances数组中则则说明它有是单例共享型，可以直接返回。
2 通过`getConcrete($abstract)` 获取`$concrete ` 实体。来看`getConcrete` 源码。

```php
 protected function getConcrete($abstract)
    {
	    //首先判断$abstact是否有上下文相关的实体
        if (! is_null($concrete = $this->getContextualConcrete($abstract))) {
            return $concrete;
        }

        // If we don't have a registered resolver or concrete for the type, we'll just
        // assume each type is a concrete name and will attempt to resolve it as is
        // since the container should be able to resolve concretes automatically.
        if (! isset($this->bindings[$abstract])) {
            return $abstract;
        }

        return $this->bindings[$abstract]['concrete'];
    }
```

检测`$this->bindings` 数组中是否存在`$abstract` ，存在返回`['concrete']` 内容，不存在则返回`$abstract`。

回到`make()` 函数，接下来要执行：

```php
if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete, $parameters);
        } else {
            $object = $this->make($concrete, $parameters);
        }

protected function isBuildable($concrete, $abstract)
    {
	    //如果$concrete和$abstract完全一样或者$concrete是个闭包函数则认为是可实例化的。
        return $concrete === $abstract || $concrete instanceof Closure;
    }
```
如果可实例化，则进行实例化，否则，则说明有依赖，需要继续递归调用 `make` 直至没有依赖。然后们跟进 `build()` 函数。

```php
public function build($concrete, array $parameters = [])
    {
        // If the concrete type is actually a Closure, we will just execute it and
        // hand back the results of the functions, which allows functions to be
        // used as resolvers for more fine-tuned resolution of these objects.
        // 如果是闭包就直接实例化。
        if ($concrete instanceof Closure) {
            return $concrete($this, $parameters);
        }
        
		//拿到一个反射实例
        $reflector = new ReflectionClass($concrete);

        // If the type is not instantiable, the developer is attempting to resolve
        // an abstract type such as an Interface of Abstract Class and there is
        // no binding registered for the abstractions so we need to bail out.
        if (! $reflector->isInstantiable()) {
            if (! empty($this->buildStack)) {
                $previous = implode(', ', $this->buildStack);

                $message = "Target [$concrete] is not instantiable while building [$previous].";
            } else {
                $message = "Target [$concrete] is not instantiable.";
            }

            throw new BindingResolutionException($message);
        }

		//入栈
        $this->buildStack[] = $concrete;

		//获取构造函数
        $constructor = $reflector->getConstructor();
        
        // If there are no constructors, that means there are no dependencies then
        // we can just resolve the instances of the objects right away, without
        // resolving any other types or dependencies out of these containers.
        // 构造函数为空，无依赖直接 new。
        if (is_null($constructor)) {
            array_pop($this->buildStack);

            return new $concrete;
        }

		//获取依赖参数 $dependencies 
        $dependencies = $constructor->getParameters();

        // Once we have all the constructor's parameters we can create each of the
        // dependency instances and then use the reflection instances to make a
        // new instance of this class, injecting the created dependencies in.
        $parameters = $this->keyParametersByArgument(
            $dependencies, $parameters
        );
        
        // 解析 $parameters 中的依赖
        $instances = $this->getDependencies(
            $dependencies, $parameters
        );
        
        array_pop($this->buildStack);

        return $reflector->newInstanceArgs($instances);
    }
```

对于 `$dependencies` 、 `$parameters` 参数。我的理解是 `$dependencies` 是解析出来的依赖（必须的），`$parameters` 是自定义的参数（可通过自定义传入来改变特定依赖）。首先看第一个步骤`keyParametersByArgument($dependencies, $parameters)` 方法的实现。

```php
protected function keyParametersByArgument(array $dependencies, array $parameters)
    {
        foreach ($parameters as $key => $value) {
            if (is_numeric($key)) {
                unset($parameters[$key]);

                $parameters[$dependencies[$key]->name] = $value;
            }
        }

        return $parameters;
    }
```

这个函数做了什么？首先 `$parameters`  里的参数应当是和`$dependencies` 一一对应的（下标），那么这个函数的作用就是把``$parameters`` 的数字`$key` 替换为对应依赖的参数的 name。

由于`$parameters` 的参数可能不是全部，所以我们需要在进一步处理，于是有了这一步

	$instances = $this->getDependencies($dependencies, $parameters);

具体实现：

```php
protected function getDependencies(array $parameters, array $primitives = [])
    {
        $dependencies = [];

        foreach ($parameters as $parameter) {
            $dependency = $parameter->getClass();
            
            // If the class is null, it means the dependency is a string or some other
            // primitive type which we can not resolve since it is not a class and
            // we will just bomb out with an error since we have no-where to go.
            if (array_key_exists($parameter->name, $primitives)) {
                $dependencies[] = $primitives[$parameter->name];
            } elseif (is_null($dependency)) {
                $dependencies[] = $this->resolveNonClass($parameter);
            } else {
                $dependencies[] = $this->resolveClass($parameter);
            }
        }

        return $dependencies;
    }
```

我们需要的依赖都被包含在传入本方法的 `$parameters` 参数中，这个参数是通过反射类型获取的依赖数组，本方法将遍历该数组的每一项，取出每一个依赖对应类的全称，先从 `$primitives` 中取以该参数为 key 的 `$value` 并放入新建的 `$dependencies` 空数组中，如果没有解析出类名则调用`resolveNonClass()` ，如果解析出类名却没有在给定参数中查找到，就调用`resolveClass()` 函数。分别来看下这俩函数做啥的。


```php
protected function resolveNonClass(ReflectionParameter $parameter)
    {
        if (! is_null($concrete = $this->getContextualConcrete('$'.$parameter->name))) {
            if ($concrete instanceof Closure) {
                return call_user_func($concrete, $this);
            } else {
                return $concrete;
            }
        }
        
        if ($parameter->isDefaultValueAvailable()) {
            return $parameter->getDefaultValue();
        }

        $message = "Unresolvable dependency resolving [$parameter] in class {$parameter->getDeclaringClass()->getName()}";

        throw new BindingResolutionException($message);
    }
```
`resolveNonClass()` 函数在解析不出类名时调用,他检测两种可能情况。

1 可能是通过上下文绑定的参数，如果是，返回。
2 可能是有默认值的参数，如果是，则返回。

关于什么是上下文绑定，以后再研究。接着来看`resolveClass()` 函数：

```php
 protected function resolveClass(ReflectionParameter $parameter)
    {
        try {
            return $this->make($parameter->getClass()->name);
        }

        // If we can not resolve the class instance, we will check to see if the value
        // is optional, and if it is we will return the optional parameter value as
        // the value of the dependency, similarly to how we do this with scalars.
        catch (BindingResolutionException $e) {
            if ($parameter->isOptional()) {
                return $parameter->getDefaultValue();
            }

            throw $e;
        }
    }
```

`resolveClass()` 函数用于处理解析出类名但不存在于自定义参数中的情况，处理方式很简单，没有传入的类名，让 `make` 帮忙。 : )

