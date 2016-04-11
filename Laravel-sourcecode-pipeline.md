title: Laravel 源码分析 -- 流水线
date: 2016-04-11 21:59
tags: [Laravel,PHP]
categories: Laravel
---

在梳理流程时遇到多次流水线操作，当时只知是用来执行中间件，现在追过去搞懂它如何实现。

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

### send($request)

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

### through()

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

本函数用于接收中间件数组，或者多个中间件的参数同时传入，并保存到本实例的`pipes ` 数组内。

### then($this->dispatchToRouter())

```php
/**
     * Run the pipeline with a final destination callback.
     *
     * @param  \Closure  $destination
     * @return mixed
     */
    public function then(Closure $destination)
    {
    //以`$passable`为参数的闭包，调用`$firstSlice()`将执行`call_user_func($destination, $passable);`
        $firstSlice = $this->getInitialSlice($destination);
        
        //反转，倒序
        $pipes = array_reverse($this->pipes);
        
        return call_user_func(
            array_reduce($pipes, $this->getSlice(), $firstSlice), 
            $this->passable
        );
    }
```

#### 1 Step

```php

$firstSlice = $this->getInitialSlice($destination);
```

第一句比较好理解，注释说的很明白，这里看一 `getInitialSlice($destination)` 下实现就 Ok。



```php
/**
     * Get the initial slice to begin the stack call.
     *
     * @param  \Closure  $destination
     * @return \Closure
     */
    protected function getInitialSlice(Closure $destination)
    {
        return function ($passable) use ($destination) {
            return call_user_func($destination, $passable);
        };
    }
```

#### 2 Step

```php
$pipes = array_reverse($this->pipes);
```

第二句将传入的中间件的数组顺序翻转。为什么要反转？暂时不管，先看第三句吧。

#### 3 Step

比较绕的是第三句。

```php
return call_user_func(
            array_reduce($pipes, $this->getSlice(), $firstSlice), 
            $this->passable
        );
```

有些懵，不怕，拆开来看。

    array_reduce($pipes, $this->getSlice(), $firstSlice)

首先要搞懂 `array_reduce` 的原理，这里直接引用文档：

    mixed array_reduce ( array $input , callable $function [, mixed $initial = NULL ] )

>用回调函数迭代地将数组简化为单一的值。

通俗些：迭代执行传入的函数 `$function`，每次的执行结果作为下一次迭代的一个参数。`initial ` 参数如果被传入将作为第一次被迭代时的参数。

