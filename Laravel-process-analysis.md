title: Laravel 流程分析
date: 2016-04-10 19:32
tags: [Laravel,PHP]
categories: Laravel
---

我是不读源码会死星人，不清楚实际运行流程会别扭，缺乏掌控感。

<!-- more -->

---

# 开始的开始。


```php
//一切都从public/index.php开始

//阶段一：加载 composer
require __DIR__ . '/../bootstrap/autoload.php';

//阶段二：实例化 Application 实例
$app = require_once __DIR__ . '/../bootstrap/app.php';

//阶段三：实例化 kernel 类，处理请求
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
	$request = Illuminate\Http\Request::capture()
);

//阶段四：返回结果
$response->send();

$kernel->terminate($request, $response);
```

# 阶段二：实例化 Application 实例

看`app.php`源码。

```php

$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);

//共享型  $this->bind($abstract, $concrete, true); 
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;
```

这一阶段实例化了容器类：`Illuminate\Foundation\Application`，并创建了单例模式的 Kernel 和 Exception 实例。我们继续跟进`Illuminate\Foundation\Application` 构造函数：

```php
public function __construct($basePath = null)
    {
	    //
        $this->registerBaseBindings();

		//
        $this->registerBaseServiceProviders();

		//
        $this->registerCoreContainerAliases();

        if ($basePath) {
            $this->setBasePath($basePath);
        }
    }
```

