---
title: PHPUnit 和 Dusk
date: 2020-09-10 19:46:53
tags: PHP,PHPUnit,Docker
categories: PHP
---

需要用 Dusk 来进行完整的功能测试，所以有了这篇探索文章。

---

# 初始化
## 接入 Dusk
- 安装 chromedriver：`php artisan dusk:chrome-driver 83`
- 安装 chromium：`apt-get update && apt-get install -y chromium`

`php artisan dusk:install` 会安装最新的 driver，但可能和当前系统不匹配，导致报错。所以需要事先通过 `chromium --version` 查看版本号，目前在我在使用的版本是 83。

https://learnku.com/docs/laravel/7.x/dusk/7512
### 执行
`php artisan dusk`、`./vendor/bin/phpunit` 都可以触发 dusk 测试。

如果遇到了 `invalid session id` 的情况，一般是在 docker 的 /dev/shm 空间不足导致，默认是 64M，有两种方案可以解决：

- 给 docker 容器分配更大的 shm-size。compose 文件中增加 `shm_size: "512M"`。
- 只启动命令中加入 `--disable-dev-shm-usage`，这个没有验证过，后来无法复现该故障了。

```PHP
There was 1 error:

1) Tests\Browser\SchoolTest::testIndex
Facebook\WebDriver\Exception\InvalidSessionIdException: invalid session id

/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/Exception/WebDriverException.php:107
/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/Remote/HttpCommandExecutor.php:370
/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/Remote/RemoteWebDriver.php:590
/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/Remote/RemoteExecuteMethod.php:27
/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/WebDriverOptions.php:166
/Users/fengit/workspace/php/Private/vendor/laravel/dusk/src/Browser.php:396
/Users/fengit/workspace/php/Private/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:161
/Users/fengit/workspace/php/Private/vendor/laravel/framework/src/Illuminate/Support/Traits/EnumeratesValues.php:202
/Users/fengit/workspace/php/Private/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:162
/Users/fengit/workspace/php/Private/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:78
/Users/fengit/workspace/php/Private/tests/Browser/SchoolTest.php:29

Caused by
Facebook\WebDriver\Exception\UnknownErrorException: unknown error: session deleted because of page crash
from unknown error: cannot determine loading status
from tab crashed
  (Session info: headless chrome=83.0.4103.116)

/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/Exception/WebDriverException.php:139
/Users/fengit/workspace/php/Private/vendor/php-webdriver/webdriver/lib/Remote/HttpCommandExecutor.php:370
/Users/fengit/workspace/php/Private/vendor/php-webdriver/webd
```

如果执行时遇到这个问题，可能需要在 chrome 的启动参数中增加这个参数 `--no-sandbox`，参考文档：https://github.com/SeleniumHQ/selenium/issues/4961

```php
[09:09:37] 1) Tests\Browser\SchoolTest::testIndex
[09:09:37] Facebook\WebDriver\Exception\UnknownServerException: unknown error: Chrome failed to start: exited abnormally
[09:09:37]   (unknown error: DevToolsActivePort file doesn't exist)
[09:09:37]   (The process started from chrome location /usr/bin/chromium is no longer running, so ChromeDriver is assuming that Chrome has crashed.)
[09:09:37]   (Driver info: chromedriver=2.45.615279 (12b89733300bd268cff3b78fc76cb8f3a7cc44e5),platform=Linux 3.10.0-1062.18.1.el7.x86_64 x86_64)
[09:09:37] 
[09:09:37] /root/workspace/anyishou_private_934D/vendor/php-webdriver/webdriver/lib/Exception/WebDriverException.php:175
[09:09:37] /root/workspace/anyishou_private_934D/vendor/php-webdriver/webdriver/lib/Remote/HttpCommandExecutor.php:376
[09:09:37] /root/workspace/anyishou_private_934D/vendor/php-webdriver/webdriver/lib/Remote/RemoteWebDriver.php:136
[09:09:37] /root/workspace/anyishou_private_934D/tests/DuskTestCase.php:40
[09:09:37] /root/workspace/anyishou_private_934D/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:200
[09:09:37] /root/workspace/anyishou_private_934D/vendor/laravel/framework/src/Illuminate/Support/helpers.php:404
[09:09:37] /root/workspace/anyishou_private_934D/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:201
[09:09:37] /root/workspace/anyishou_private_934D/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:95
[09:09:37] /root/workspace/anyishou_private_934D/vendor/laravel/dusk/src/Concerns/ProvidesBrowser.php:65
[09:09:37] /root/workspace/anyishou_private_934D/tests/Browser/SchoolTest.php:23
```

数据库相关问题
### 一些探索
- dusk 无法使用 in-memory 的 sqlite。

配置文件怎么配置？
看命令，如果是
## 接入 Dusk Dashboard
### 安装
安装时比较消耗 CPU，如果出现 Killed 时，考虑提高 CPU 限制。

`comoser require -dev beyondcode/dusk-dashboard`

### 运行
执行`php artisan dusk:dashboard`时出现如下报错：

```
stream_set_blocking() expects parameter 1 to be resource, null given
```

是因为方法被禁用了，调整 php.ini 文件的中的`disable_functions`即可。
### 访问
##### 直接暴露端口到宿主机
##### nginx 反向代理，通过 80 端口访问。

dashboard 后端出于安全考虑会检测 host，所以反向代理时需要指定`Host`


```nginx
server {
    listen   80;
    server_name private-dusk.local.tbxzs.cn;
    
    location /socket {
        proxy_pass http://__DOCKER_PHP_FPM_74__:9773/socket;
        # this magic is needed for WebSocket
        proxy_http_version  1.1;
        proxy_set_header    Upgrade $http_upgrade;
        proxy_set_header    Connection "upgrade";
        proxy_set_header    Host "private.local.tbxzs.cn:9773";
        proxy_set_header    X-Real-IP $remote_addr;
    }
    
    location / {
          proxy_pass http://__DOCKER_PHP_FPM_74__:9773;
          proxy_set_header    Host "private.local.tbxzs.cn:9773";
     }
}
```