title: Laravel 源码分析 -- 流水线
date: 2016-04-11 21:59
tags: [Laravel,PHP]
categories: Laravel
---

流水线操作在梳理流程时遇到过多次，当时只知道是用来执行中间件的，现在追过去把它怎么执行的搞懂。

```php
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
```

<!-- more -->

---

