title: Laravel 源码分析 -- 流水线
date: 2016-04-11 21:59
tags: [Laravel,PHP]
categories: Laravel
---

在梳理流程时多次遇到流水线操作，当时只知是用来执行中间件，现在去搞懂它如何实现。

```php
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
```

<!-- more -->

---

### 从构造函数`new Pipeline($this->app)`开始

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

构造函数接收了`application` 容器实例并保存。

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
	    //以`$passable`为参数的闭包，调用`$firstSlice()`将执行`call_user_func($destination, $passable);` 并返回执行结果。
        $firstSlice = $this->getInitialSlice($destination);
        
        //反转，倒序
        $pipes = array_reverse($this->pipes);
        
        return call_user_func(
            array_reduce($pipes, $this->getSlice(), $firstSlice), 
            $this->passable
        );
    }
```

这一段是实现重点，我们一步一步看。

#### 1 Step

```php
$firstSlice = $this->getInitialSlice($destination);
```

第一句比较好理解，注释说的很明白。 `$firstSlice`函数用于执行传入的 `$destination` 闭包。这里简单看一下 `getInitialSlice($destination)` 的实现：



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

第二句将传入的中间件的数组顺序翻转。为什么要反转？暂时不管，看第三句。

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

通俗些：迭代执行传入的函数 `$function`，每次的执行结果作为下一次迭代的参数之一。`initial ` 参数如果被传入将作为第一次被迭代时的参数。

回到这句话来看

        array_reduce($pipes, $this->getSlice(), $firstSlice)

这里不明晰的只有`$this->getSlice()` 参数，跟进：

```php
/**
     * Get a Closure that represents a slice of the application onion.
     *
     * @return \Closure
     */
    protected function getSlice()
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                // If the pipe is an instance of a Closure, we will just call it directly but
                // otherwise we'll resolve the pipes out of the container and call it with
                // the appropriate method and arguments, returning the results back out.
                if ($pipe instanceof Closure) {
                    return call_user_func($pipe, $passable, $stack);
                } else {
                    list($name, $parameters) = $this->parsePipeString($pipe);

                    return call_user_func_array([$this->container->make($name), $this->method],
                            array_merge([$passable, $stack], $parameters));
                }
            };
        };
    }
```

慢着，我仿佛看到无尽的闭包和`call_user_func` ，我想静静..

哈哈，不着急，一点点捋，执行`getSlice()` 我们将得到要迭代的函数：`function ($stack, $pipe)` 。迭代执行一次将返回一个闭包 `function ($passable) use ($stack, $pipe)` ，这个闭包将作为参数 `$stack` 传入下一次迭代中，当执行到最后一次迭代时，我们得到的最终函数是（我们用f（n）表示本次迭代返回值）：

	function ($passable) use (f(n-1), $middleware[n-1]->handle());


也许依然懵懵的，别着急，接着往下看，`array_reduce`最终的迭代结果参与了`call_user_func` 的运行。

```php
return call_user_func(
            array_reduce($pipes, $this->getSlice(), $firstSlice), 
            $this->passable
        );
```

通过`call_user_func` ，我们将调用最终迭代返回的闭包。

```php
function ($passable) use ($stack, $pipe) {
                // If the pipe is an instance of a Closure, we will just call it directly but
                // otherwise we'll resolve the pipes out of the container and call it with
                // the appropriate method and arguments, returning the results back out.
                if ($pipe instanceof Closure) {
                    return call_user_func($pipe, $passable, $stack);
                } else {
                    list($name, $parameters) = $this->parsePipeString($pipe);
                    
                    print_r($this->method);die();

                    return call_user_func_array([$this->container->make($name), $this->method],
                            array_merge([$passable, $stack], $parameters));
                }
            };
```

它将执行语句：

    call_user_func($pipe, $passable, $stack);

我们以`$passable` 、`$stack` 为参数，调用了`$pipe` 函数，`$pipe` 函数是啥来着？ 是中间件的 handle() 实现：

```php
/**
    public function handle($request, Closure $next, $guard = null)
    {
	    //dosomething...

        return $next($request);
    }
```

`handle()` 函数接收了`$request` 并进行相关处理，然后交由传进入的第二个闭包做后续操作。第二参数是我们刚传进来的`$stack`  ，它是我们在执行array_reduce 时倒数第二次迭代的结果。`$stack`函数将再次调用指定中间件的`handle()` 来对`$request` 处理，处理后将结果继续交给传给它的`$stack` ，以此类推，直至将加工好的`$request` 交给第一次迭代时传入的初始化参数 `$firstSlice` 闭包，这个闭包是将`$request` 作为参数传递给`then()` 指定的函数，比如路由分发。

现在知道第二部为什么要倒叙了，像压栈一样，我们要先把最后需要执行的闭包作为参数，放进去。先进，后执行。这也是迭代结果的参数名叫 `$stack` 的原因吧，很形象。

至此，就实现了按照中间件的顺序对`$request` 处理，然后结果交由下家做后续处理的流程。

的确有些像流水线。

