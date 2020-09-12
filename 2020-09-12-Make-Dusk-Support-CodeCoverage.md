---
title: Dusk 支持代码覆盖率监测
date: 2020-09-12 21:44:38
tags: Dusk,PHPUnit,CodeCoverage
categories: Dusk
---

## 触发额外的一份覆盖率监测
在 Dusk 中触发 Code Coverage 的方法 [issue #258](https://github.com/laravel/dusk/issues/258)，示例如下，在 build/dusk 生成 cov 文件后，再使用 [phpcov](https://github.com/sebastianbergmann/phpcov) 去合并单元测试。
```php
    public function register()
    {
        if ($this->app->environment('production')) {
            throw new Exception('It is unsafe to run Dusk in production.');
        }
        if (!$this->app->runningInConsole()) {
            if ($this->app->environment('testing')) {
                try {
                    $this->triggerCoverage();
                } catch (Exception $e) {
                    Log::info("Dusk coverage: " . $e->getMessage());
                }
            }
        }
    }

    private function triggerCoverage() {
        $coverage = new \SebastianBergmann\CodeCoverage\CodeCoverage();
        $coverage->filter()
            ->addDirectoryToWhitelist(app_path());
        $name = microtime(true) . '.' . str_random(8);
        $coverage->start($name);
        $this->app->terminating(function () use ($coverage, $name) {
            $coverage->stop();
            $writer = new \SebastianBergmann\CodeCoverage\Report\PHP();
            $dir = base_path() . "/build/dusk";
            if (!file_exists($dir)) mkdir($dir, 0755, true);
            $writer->process($coverage, "{$dir}/{$name}.php.cov");
        });
    }
```

## 合并覆盖率报告（patch）、生成新的 html
用 phpcov 合并覆盖率检测文件：[地址](https://medium.com/@at_ishikawa/show-test-coverage-reports-using-phpunit-6e51230eb3a2)，示例：
```
> vendor/bin/phpunit --coverage-php coverage/SampleTest.cov
 tests/SampleTest.php
PHPUnit 7.5.16 by Sebastian Bergmann and contributors.
.                                                                   1 / 1 (100%)
Time: 53 ms, Memory: 4.00 MB
OK (1 test, 1 assertion)
Generating code coverage report in PHP format ... done


> vendor/bin/phpunit --coverage-php coverage/Sample2Test.cov tests/Sample2Test.php
PHPUnit 7.5.16 by Sebastian Bergmann and contributors.
.                                                                   1 / 1 (100%)
Time: 24 ms, Memory: 4.00 MB
OK (1 test, 2 assertions)
Generating code coverage report in PHP format ... done


> vendor/bin/phpcov merge coverage --html report
phpcov 5.0.0 by Sebastian Bergmann.
Generating code coverage report in HTML format ... done
```

其他实现参考：
- [Code coverage for laravel dusk](https://stackoverflow.com/questions/43653881/code-coverage-for-laravel-dusk)
- [coverage_for_dusk.php](https://gist.githubusercontent.com/jleonardolemos/3599ae185ab1bc6f3901d5881f977fd6/raw/91de0098388769db035484dbf92ae6393cff8ba9/coverage_for_dusk.php)


## 问题
执行`phpcov merge`时，出现了以下报错，经排查，是最近才被解决的问题，需要更新库到最新版：`composer require --dev 'phpunit/php-code-coverage:^8.0'`，升级库有些难度，待处理。相关 [issue](https://github.com/Codeception/Codeception/issues/5949)
```php
PHP Fatal error: Uncaught ArgumentCountError: Too few arguments to function SebastianBergmann\CodeCoverage\CodeCoverage::__construct(), 0 passed in ./vendor/codeception/codeception/src/Codeception/Coverage/Subscriber/Printer.php on line 37 and exactly 2 expected in ./vendor/phpunit/php-code-coverage/src/CodeCoverage.php:118
```
