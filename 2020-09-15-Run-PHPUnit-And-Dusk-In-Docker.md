---
title: Dusk 中进行单元测试（含 Dusk）整体流程
date: 2020-09-11 19:37:02
tags: PHPUnit,Dusk,Chrome
categories: PHPUnit
---

注意事项：
- 以下命令需要在项目对应的 php 容器中执行，开发人员自行根据 php 版本来决定。新项目一般都是 **7.4** 版本的 php，所以选 `php-fpm74-dev` 对应的容器即可。
- 目前在运转的单元测试流水线，使用的也是本文档中的命令和镜像，所以请开发人员在推送代码前，**务必**在本地执行成功后再推送到线上，否则流水线会报错。
- 测试使用的数据库需要使用本地的 sqlite 文件：database/database.sqlite，如果不存在请自行创建。不再使用内存类型的 sqlite。（下文的配置文件示例中会对数据库指定，按照模板操作即可。）
- 测试流程中，所有环境变量均从 `.env.testing`、`phpupnit.xml` 中读取。（下文的命令中会强行指定配置文件，按照命令操作即可。）
    - 注意：`.env.testing` 中的下参数固定：`APP_URL=http://localhost`
- 暂定只用 phpunit 来启动测试流程，不再使用 `php artisan dusk`。
- 请尽量参考本教程去实现开发流程，尤其是`代码覆盖率检测`，请务必参考本教程实现，因为 dusk 相关的测试代码原本不支持覆盖率检测，需要做了一些微调。

下面测试流程开始：

#### 进入容器内的项目目录
```
docker exec -it 容器名或容器 ID 前 3 位 bash
cd 项目目录
```

#### 初始化项目
第一次 git clone 项目代码后，执行以下命令，后续不再需要执行。

```
# 安装类库
composer install && composer du

# 指定浏览器驱动的版本。
php artisan dusk:chrome-driver 83

# 创建本地 sqlite 文件。
touch database/database.sqlite
```

phpunit.xml 配置示例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="./vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
>
    <testsuites>
        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
        </testsuite>
        <testsuite name="Browser">
            <directory suffix="Test.php">./tests/Browser</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">./app</directory>
            <exclude>
                <directory suffix="routes.php">./app/*</directory>
            </exclude>
        </whitelist>
    </filter>
    <php>
        <server name="APP_ENV" value="testing"/>
        <server name="BCRYPT_ROUNDS" value="4"/>
        <server name="CACHE_DRIVER" value="array"/>
        <server name="DB_CONNECTION" value="sqlite"/>
        <server name="MAIL_MAILER" value="array"/>
        <server name="QUEUE_CONNECTION" value="sync"/>
        <server name="SESSION_DRIVER" value="array"/>
        <server name="TELESCOPE_ENABLED" value="false"/>
    </php>
    <logging>
        <log type="coverage-html" target="./tests/codeCoverage/html" charset="UTF-8"/>
        <log type="coverage-php" target="./tests/codeCoverage/cov/console.cov" charset="UTF-8"/>
    </logging>
</phpunit>
```

.env.test 基本配置示例：
```
APP_NAME=Laravel
APP_ENV=testing
APP_KEY=base64:byUfqtk2LvEK4HYfYSfOC8M8/rt610qKjQHjWHMOR4E=
APP_DEBUG=true

APP_URL=http://localhost
DUSK_DASHBOARD_URL=http://localhost
DB_CONNECTION=sqlite
LOG_CHANNEL=stack

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

#### 启动 Laravel 本地服务
```
php artisan serve --port 80 --env=testing &
```
- 启动本地服务的目的是为了摆脱测试流程对 nginx 的依赖。
- 每次重启容器后，都要执行以下命令，因为本地服务不自启。

#### 执行单元测试。

```
./vendor/bin/phpunit
```

#### 生成代码覆盖率报告
```
/root/.composer/vendor/bin/phpcov merge tests/codeCoverage/cov/ --html tests/codeCoverage/html/
```
- 报告存放的目录是：`tests/codeCoverage/html/`
- codeCoverage 目录不用上传到 git。

#### 使用 dusk:dashboard
```
php artisan dusk:dashboard --env=testing
```

- 执行本条命令后，dashboard 的入口为：http://localhost:9773/dashboard 。 如果浏览器报 404，说明配置文件中 APP_URL 没有指定为 http://localhost。
- dashboard 对 dusk 的支持并不完善，有些请求会因为页面内容过多不显示在测试列表里，进而导致 dashboard 无响应。实际上测试流程可能已经测试通过了，所以不用过度依赖 dashboard。针对这类看不到的页面如果想查看页面，可以考虑 screenshot、dump 方法。

### 代码规范检测教程