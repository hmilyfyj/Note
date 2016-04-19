title: Laravel 问题集锦
date: 2016-04-19 08:32
tags: [Laravel,PHP]
categories: Laravel
---

<!-- more -->

---

### 1. Undefined variable: errors

#### 原因

在老版本中就算用户不向模版中传送 `$errors` 参数，Laravel 也会设置一个 `$errors` 参数，它的生成是由中间件：`\Illuminate\View\Middleware\ShareErrorsFromSession::class` 生成的。

然而在 v5.2 版本后，该中间件不再伴随着每一个请求，而是被分入了 `$middewareGroups` 数组的 `web` 字段内，即：

```php
protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
        ],

        'api' => [
            'throttle:60,1',
        ],
    ];
```

#### 解决

添加 `web` 中间件。如：

```php
Route::group(['middleware' => ['web']], function () {

});
```

