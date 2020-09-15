---
title: PHPUnit 和 Dusk
date: 2020-09-10 19:46:53
tags: PHP,PHPUnit,Docker
categories: PHP
---
需要用 Dusk 来进行完整的功能测试，所以有了这篇探索文章。
---

# 初始化

- 如果需要单独执行 dusk，没有 nginx 作为代理，需要先执行 `php artisan serve --env=dusk.testing` 可以指定变量文件为：`.env.dusk.enving`。
- `php artisan dusk` 同样可以使用上述方法指定环境变量。
- `phpunit` 不确定。
- 使用 dusk 时，自动读取 phpunit.dusk.xml。
- 使用 phpunit 时，自动读取 phpunit.xml。

## 接入 Dusk
- 安装 dusk：https://learnku.com/docs/laravel/7.x/dusk/7512
- 安装 chromium：`apt-get update && apt-get install -y chromium`
- 安装 chromedriver：`php artisan dusk:chrome-driver 83`
`php artisan dusk:install` 会安装最新的 driver，最新版可能和当前系统内的 Chrome 版本不匹配，进而导致报错。所以在安装前可以先通过 `chromium --version` 查看版本号，用于制定 driver 的版本号，目前在我在使用的版本是 83，最新版是 85，所以在安装 driver 的命令中指定版本号为 83。
### 执行
`php artisan dusk`、`./vendor/bin/phpunit` 都可以触发 dusk 测试。

#### 报错：invalid session id
如果遇到了 `invalid session id` 的情况，一般是在 docker 的 /dev/shm 空间不足导致，默认是 64M，有两种方案可以解决：
- 给 docker 容器分配更大的 shm-size。compose 文件中增加 `shm_size: "512M"`。
- 在启动命令中加入 `--disable-dev-shm-usage`，这个没有验证过，后来无法复现该故障了。

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
#### 报错：Chrome has crashed
如果执行时遇到`Chrome has crashed`的报错，可尝试在 Chrome 的启动参数中增加这个参数 `--no-sandbox`，参考文档：https://github.com/SeleniumHQ/selenium/issues/4961，网页中还提到了另一个参数，暂未验证：`--single-process`

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
#### 报错：Authenticatable 抛异常
```php
Argument 1 passed to Illuminate\Auth\SessionGuard::login() must implement interface Illuminate\Contracts\Auth\Authenticatable, null given, called in /home/wwwroot/Anyishou/vendor/laravel/dusk/src/Http/Controllers/UserController.php on line 47
```
这是登录失败触发的故障，原因可能是表里没数据或者根本没有建表，需要关注接口报错日志，此时接口的报错不会显示到 dusk 里，所以不要只通过 dusk 控制台排查。通过 sentry 日志可知具体报错。

![企业微信截图_1230d378-07f0-4a82-82a5-8cb6de562869](/media/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1230d378-07f0-4a82-82a5-8cb6de562869.png)
数据库相关问题
### 一些探索
- dusk 无法使用 in-memory sqlite，所以使用本地数据库，方法：在 database 目录下新建空文件即可解决问题。
- Dusk 测试用例中，要使用 use DatabaseMigrations 来构建数据，而不是 RefreshDatabase。
- 牵涉到页面跳转的，慎重使用 pressAndWaitfor，除非你很了解这个方法的作用。
- 牵涉到多次登陆的，记得及时清理 cookie 或者全程登录一个账号。

配置文件怎么配置？当执行 `phpunit` 命令时，`dusk` 会直接读取 .env 的内容，所以跑测试时，页面会输出 500。当执行 `php atrisan dusk` 时，dusk 会优先读取 `.env.dusk` 的配置信息。至于 `.env.testing` 这类配置方法，我没有找到利用的方法，配置后不生效，所以暂不考虑处理。

所以，关于执行测试的方案有两个：
1. 只用 dusk 跑测试。
2. 同时使用 phpunit、dusk，分别用于普通测试和浏览器测试。
3. 只用 phpunit 跑测试，数据库使用本地的 sqlite，测试时，构建 .env 文件，这样也可以执行 dusk 测试。
## 接入 Dusk Dashboard
### 安装
安装命令：`comoser require -dev beyondcode/dusk-dashboard`
安装时比较消耗 CPU，如果出现 Killed 时，考虑提高 CPU 限制，如果是在 docker 内执行，提升 cpu 的方法参考另一篇文章。
### 运行
执行`php artisan dusk:dashboard`时出现如下报错：
```
stream_set_blocking() expects parameter 1 to be resource, null given
```
这是因为 PHP 运行环境出于安全考虑禁用了该方法，调整 php.ini 文件的中的`disable_functions`字段即可。
### 访问 dashborad
##### 直接暴露端口到宿主机
##### nginx 反向代理，通过 80 端口访问。
dashboard 应该是出于安全考虑，会检测 host 是否为 env 文件中指定 APP_URL，所以在反向代理时需要指定`Host`，否则访问页面时会 404。
```nginx
server {
    listen   80;
    server_name dusk.local;
    
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