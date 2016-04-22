title: Laravel 源码分析（七） -- 自带用户校验的实现 （未完）
date: 2016-04-19 15:31
tags: [Laravel,PHP]
categories: Laravel
---

读  `Laravel` 源码有一段时间了，那么这些功能是如何发挥作用的呢？

今天看 `Laravel` 官方如何实现自带的校验、注册流程。

<!-- more -->

---

# 新知识点

Trait

# 创建路由

```php
Route::group(['middleware' => ['web']], function () {
	Route::auth();
});
```

在这里说一下，原先用默认的命令创建的路由是这样的：

```php
Route::auth();

Route::group(['middleware' => ['web']], function () {
});
```

这种情况会产生报错 `Undefined variable: errors` 错误，原因在上一篇  [错误集锦](/2016/04/19/Laravel-normal-problems/)  提过。

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


`Auth` 类是通过门面 `Facade` 实现的，采用静态方法调用 `AuthManager` （类别名 `auth`）实例的各个方法，如 `Auth::guard($guard)`，该方法将返回一个 `SessionGuard` ， “Session 警卫”  的职责就是检测 访问用户的权限 、操纵用户的 `Session` 等。

那么，这个 `auth` 中间件的作用就出来了：检测 用户权限，访客则跳转登陆界面（或返回未授权信息）、用户则继续下一步处理。

# 登陆

用户提交的表单交给 `AuthController` 的 `login()` 方法：

```php
public function login(Request $request)
    {
        // 1. 验证请求。（验证方法单写一篇笔记。）
        $this->validateLogin($request);

        // If the class is using the ThrottlesLogins trait, we can automatically throttle
        // the login attempts for this application. We'll key this by the username and
        // the IP address of the client making these requests into this application.
        // 判断是本类否使用了 `ThrottlesLogins`。 用于防范暴力破解。
        $throttles = $this->isUsingThrottlesLoginsTrait();

        // 请求次数超过限制，触发压制事件
        if ($throttles && $lockedOut = $this->hasTooManyLoginAttempts($request)) {
            $this->fireLockoutEvent($request);

            return $this->sendLockoutResponse($request);
        }
        
        // 获取用户提交的信息
        $credentials = $this->getCredentials($request);
        
        // 尝试登录
        if (Auth::guard($this->getGuard())->attempt($credentials, $request->has('remember'))) {
            // 登陆成功，返回登陆结果。
            return $this->handleUserWasAuthenticated($request, $throttles);
        }

        // If the login attempt was unsuccessful we will increment the number of attempts
        // to login and redirect the user back to the login form. Of course, when this
        // user surpasses their maximum number of attempts they will get locked out.
        if ($throttles && ! $lockedOut) {
            // 登录失败，增加尝试次数。
            $this->incrementLoginAttempts($request);
        }

        // 返回失败信息。
        return $this->sendFailedLoginResponse($request);
    }
```

## login() 流程

注释在代码内补上。

1. 验证请求。（验证方法单写一篇笔记。）
2. 判断是本类否使用了 `ThrottlesLogins`。
这个实现很有意思，要说一下：

```php
protected function isUsingThrottlesLoginsTrait()
    {
        return in_array(
            ThrottlesLogins::class, class_uses_recursive(static::class)
        );
    }

function class_uses_recursive($class)
    {
        $results = [];

        foreach (array_merge([$class => $class], class_parents($class)) as $class) {
            $results += trait_uses_recursive($class);
        }

        return array_unique($results);
    }
    
    
function trait_uses_recursive($trait)
    {
        $traits = class_uses($trait);

        foreach ($traits as $trait) {
            $traits += trait_uses_recursive($trait);
        }

        return $traits;
    }
```

`class_uses_recursive` 实现：

`class_uses_recursive()` 函数将遍历传入类及其父类的子类，然后通过 `trait_uses_recursive()` 方法遍历其 use 过的 Trait ，并放入数组中返回。


# 与CI比较

这些功能在 `CI` 里较难实现，一是因为 `CI` 虽然有 `Hooks`， 但 `Hooks` 可执行的地方并不能精细到这种地步 和 阶段（我记得 `hooks` 在某 `controller` 之前执行的话，基本上啥都没加载，就不要提权限啥的，那时 `Session` 类库都得不到。），再就是 `CI` 没有路由控制，我们没有办法分组指定一组进行权限控制，这样控制起来会有些麻烦。  

当然，更大的原因可能是我用 `CI` 不精。 

# 参考资料

这里有个参考资料。

[V2ex](http://v2ex.com/t/272328#reply34)

