
title: Laravel 源码分析（八） -- View 的渲染 (未完)
date: 2016-4-20 21:56
tags: [Laravel,PHP]
categories: Laravel
---

前一篇笔记讲过 [ 路由如何进行路由的匹配、分发 ](http://b.fengbl.cn/2016/04/15/Laravel-sourcecode-how-route-work/#%E5%8C%B9%E9%85%8D%E7%BB%93%E6%9E%9C%E5%A4%84%E7%90%86) ，其中最后一步就是对路由返回的结果进行包装处理。有时路由会引入模版，如：

```php
Route::group(['middleware' => ['web']], function () {
	Route::get('/test', [ function () {
		return view('auth.login');
	}]);
});
```

那么 `view()` 函数如何执行？返回的是什么内容？模版内容又在哪里被渲染的呢？本篇笔记就是要搞清这个。


<!-- more -->

---

首先，`view()` 是个全局函数，它在 `/laravel/framework/src/Illuminate/Support/helpers.php` 文件内定义，该文件由 `composer` 在项目启动初期就执行，由 `autoload_real.php` 执行，初期需要自动执行的文件列表在 `autoload_files.php` 内定义。不是本笔记重点，不再多说。

# 正文

## `view()` 的实现：

```php
function view($view = null, $data = [], $mergeData = [])
    {
        $factory = app(ViewFactory::class);

        if (func_num_args() === 0) {
            return $factory;
        }

        return $factory->make($view, $data, $mergeData);
    }
```

### 功能：

获取 `ViewFactory` 实例并调用其 `make()` 方法，然后返回执行结果。

首先要找到 `ViewFactory` 类。

## ViewFactory 

`ViewFactory`  是类 `Illuminate\Contracts\View\Factory` 的别名。这个类由第一次出现是在 `Application` 类内，并被定义了别名 `view`：

	'view' => ['Illuminate\View\Factory', 'Illuminate\Contracts\View\Factory']

复习下别名的作用：

当我们通过 `app("ClassName")` 的方法调用 `Application::make()` 方法去请求某个实例时，会调用与该类别名绑定的实例化方法。如 `view` ， 其实例化方法在 `Illuminate\View\ViewServiceProvider` 内定义。

大部分需要启动的服务提供者，都在 `config/app.php` 内的 `providers` 数组内声明，为什么要在这声明？因为它会自己动。

看 `view` ( 即 `ViewFactory` ) 服务的注册方法： 

```php
public function registerFactory()
    {
        $this->app->singleton('view', function ($app) {
            // Next we need to grab the engine resolver instance that will be used by the
            // environment. The resolver will be used by an environment to get each of
            // the various engine implementations such as plain PHP or Blade engine.
            $resolver = $app['view.engine.resolver'];
            
            $finder = $app['view.finder'];
            
            $env = new Factory($resolver, $finder, $app['events']);

            // We will also set the container instance on this view environment since the
            // view composers may be classes registered in the container, which allows
            // for great testable, flexible composers for the application developer.
            $env->setContainer($app);

            $env->share('app', $app);

            return $env;
        });
    }
```
`view` 实际上绑定了 `Illuminate\View\Factory` 类的实现，`Factory` 实例化需要三个参数，分别对应的功能是：模版解析渲染、模版文件寻找、事件触发实例。

继续来看 `Factory` 的 `make()` 方法：

```php
public function make($view, $data = [], $mergeData = [])
    {
        if (isset($this->aliases[$view])) {
            $view = $this->aliases[$view];
        }

        $view = $this->normalizeName($view);

        $path = $this->finder->find($view);

        $data = array_merge($mergeData, $this->parseData($data));

        $this->callCreator($view = new View($this, $this->getEngineFromPath($path), $view, $path, $data));

        return $view;
    }
```

`make()` 方法创建了一个 `Illuminate\View\View` 实例。这一步只是返回了一个包含 模版信息的实例，并没有进行模版的渲染，那么模版渲染是在什么时候呢？

渲染是在上一篇笔记中提到的一个过程  `prepareResponse()` 函数处理路由执行结果 时进行的：

```php
public function prepareResponse($request, $response)
    {
        if ($response instanceof PsrResponseInterface) {
            $response = (new HttpFoundationFactory)->createResponse($response);
        } elseif (! $response instanceof SymfonyResponse) {
            $response = new Response($response);
        }
        
        return $response->prepare($request);
    }
```

`View` 对象被作为参数传入 `Response` 类实例化，该类构造函数如下：

```php
public function __construct($content = '', $status = 200, $headers = array())
    {
        $this->headers = new ResponseHeaderBag($headers);
        $this->setContent($content);
        $this->setStatusCode($status);
        $this->setProtocolVersion('1.0');
    }

public function setContent($content)
    {
        if (null !== $content && !is_string($content) && !is_numeric($content) && !is_callable(array($content, '__toString'))) {
            throw new \UnexpectedValueException(sprintf('The Response content must be a string or object implementing __toString(), "%s" given.', gettype($content)));
        }

        $this->content = (string) $content;

        return $this;
    }
```

传入的 `View` 对象被作为字符串保存于本类 `$content` 参数内。从这里可以猜到，`View` 类应该实现了 `__toString` 方法，查看一下：

```php
public function __toString()
    {
        return $this->render();
    }

public function render(callable $callback = null)
    {
        try {
            $contents = $this->renderContents();
            
            $response = isset($callback) ? call_user_func($callback, $this, $contents) : null;

            // Once we have the contents of the view, we will flush the sections if we are
            // done rendering all views so that there is nothing left hanging over when
            // another view gets rendered in the future by the application developer.
            $this->factory->flushSectionsIfDoneRendering();

            return ! is_null($response) ? $response : $contents;
        } catch (Exception $e) {
            $this->factory->flushSections();

            throw $e;
        } catch (Throwable $e) {
            $this->factory->flushSections();

            throw $e;
        }
    }
```

果然，实现的 `__toString()` 函数通过 `render()` 函数对模版进行了渲染并返回了字符串格式的 `html` 内容。

具体的模版渲染过程，以后再补。

# To be coninued..

