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

我们来看第二句`$this->registerBaseBindings();` 实现：

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

        // First, we will extract the alias from the abstract if it is an array so we
        // are using the correct name when binding the type. If we get an alias it
        // will be registered with the container so we can resolve it out later.
        if (is_array($abstract)) {
            list($abstract, $alias) = $this->extractAlias($abstract);

            $this->alias($abstract, $alias);
        }

        unset($this->aliases[$abstract]);

        // We'll check to determine if this type has been bound before, and if it has
        // we will fire the rebound callbacks registered with the container and it
        // can be updated with consuming classes that have gotten resolved here.
        $bound = $this->bound($abstract);

        $this->instances[$abstract] = $instance;

        if ($bound) {
            $this->rebound($abstract);
        }
    }
```



