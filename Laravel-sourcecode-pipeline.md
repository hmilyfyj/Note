title: Laravel 源码分析 -- 流水线
date: 2016-04-11 21:59
tags: [Laravel,PHP]
categories: Laravel
---

在梳理流程时遇到多次流水线操作，当时只知是用来执行中间件，现在追过去搞懂它怎么执行的中间件。

```php
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
```

<!-- more -->

---

### 从构造函数开始

```php
 /**
     * Create a new class instance.
     *
     * @param  \Illuminate\Contracts\Container\Container  $container
     * @return void
     */
    public function __construct(Container $container)
    {
        $this->container = $container;
    }
```

传入`application` 容器实例。

### `->send($request)`

```php
/**
     * Set the object being sent through the pipeline.
     *
     * @param  mixed  $passable
     * @return $this
     */
    public function send($passable)
    {
        $this->passable = $passable;

        return $this;
    }
```

接收 `$request` 实例并保存，然后返回自身（流水线方式）。 

### ->through()

    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)

```php
/**
     * Set the array of pipes.
     *
     * @param  array|mixed  $pipes
     * @return $this
     */
    public function through($pipes)
    {
        $this->pipes = is_array($pipes) ? $pipes : func_get_args();

        return $this;
    }
```

本函数用于接收中间件数组，或者多个中间件的参数同时传入。



