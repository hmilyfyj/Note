title: Laravel 源码分析 -- 自带用户校验的实现
date: 2016-04-19 15:31
tags: [Laravel,PHP]
categories: Laravel
---

读  `Laravel` 源码有一段时间了，那么这些功能是如何发挥作用的呢？

今天看 `Laravel` 官方如何实现自带的校验、注册功能。

<!-- more -->

---

# 新知识点

Trait



# 校验状态

中间件 `auth` 实现了用户状态的检测，看其 `handle()` 函数代码；

```php
public function handle($request, Closure $next, $guard = null)
    {
        if (Auth::guard($guard)->guest()) {
            if ($request->ajax() || $request->wantsJson()) {
                return response('Unauthorized.', 401);
            } else {
                return redirect()->guest('login');
            }
        }

        return $next($request);
    }
```

`Auth` 类是通过门面 `Facade` 实现的，采用静态方法调用 `AuthManager` （类别名 `auth`）实例的各个方法：



