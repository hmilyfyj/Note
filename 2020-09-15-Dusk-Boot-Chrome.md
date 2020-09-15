---
title: 在 Dusk 中启动 Chrome
date: 2020-09-15 20:28:20
tags: PHPUnit,Dusk,Chrome
categories: PHPUnit
---

启动 Chrome 时，有两种传参方式，分别是 ChromeOptions::CAPABILITY、ChromeOptions::CAPABILITY_W3C。

其中第一种方式即将被废弃，所以查了资料，改为使用后者，两者传参方式有趣，一开始没有留意到这个地方，卡了一段时间。后来在官方 git 里找到了解决方案。这也是记录本篇笔记的原因，希望可以引以为戒。

ChromeOptions::CAPABILITY：
```php
        $options = [
            // 'debuggerAddress' => '127.0.0.1:38947',
            'args' => [
                '--no-sandbox',
                '--headless',
                '--disable-gpu',
                '--window-size=1920,1080',
                '--disable-dev-shm-usage',
            ],
        ];

        return RemoteWebDriver::create(
            'http://localhost:9515',
            DesiredCapabilities::chrome()->setCapability(
                ChromeOptions::CAPABILITY,
                $options
            )
        );
```

ChromeOptions::CAPABILITY_W3C：
```php
        $options = [
            // 'debuggerAddress' => '127.0.0.1:38947',
            'args' => [
                '--no-sandbox',
                '--headless',
                '--disable-gpu',
                '--window-size=1920,1080',
                '--disable-dev-shm-usage',
            ],
        ];

        return RemoteWebDriver::create(
            'http://localhost:9515',
            DesiredCapabilities::chrome()->setCapability(
                // ChromeOptions::CAPABILITY,
                ChromeOptions::CAPABILITY_W3C,
                $options
            )
        );
```

相关资料：
https://github.com/php-webdriver/php-webdriver/wiki/ChromeOptions
https://github.com/php-webdriver/php-webdriver/blob/d01e823a2605118f41638e7d8ebcce870ff70e1d/tests/unit/Remote/DesiredCapabilitiesTest.php
https://packagist.org/packages/php-webdriver/webdriver