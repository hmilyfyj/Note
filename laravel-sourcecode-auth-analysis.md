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


`Auth` 类是通过门面 `Facade` 实现的，采用静态方法调用 `AuthManager` （类别名 `auth`）实例的各个方法，如 `Auth::guard($guard)`，该方法将返回一个 `SessionGuard` ，这个 “Session 警卫”  的职责就是检测 访问用户的权限 、操纵用户的 `Session` 等。

那么，这个 `auth` 中间件的作用就出来了：检测 用户权限，访客则跳转、用户则继续。

这些功能在 `ci` 里较难实现，一是因为 `CI` 虽然有 `Hooks`， 但 `Hooks` 可执行的地方并不能精细到这种地步，再就是 `CI` 没有路由控制，我们没有办法分组指定一组进行权限控制，这样控制起来会有些麻烦。  

当然，更大的原因有可能是我用 `CI` 不精。 

