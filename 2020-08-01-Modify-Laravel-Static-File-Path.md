---
title: 替换 Laravel-Admin 静态资源
date: 2020-08-01 12:18:35
tags: 折腾
categories: 折腾
img: /media/Snipaste_2020-08-02_17-49-34.png
---

# 原有替换方案

修改 `\Encore\Admin\Admin.php` 文件的属性：
```php

    /**
     * @var array
     */
    public static $baseCss = [
        'vendor/laravel-admin/AdminLTE/bootstrap/css/bootstrap.min.css',
        'vendor/laravel-admin/font-awesome/css/font-awesome.min.css',
        'vendor/laravel-admin/laravel-admin/laravel-admin.css',
        'vendor/laravel-admin/nprogress/nprogress.css',
        'vendor/laravel-admin/sweetalert2/dist/sweetalert2.css',
        'vendor/laravel-admin/nestable/nestable.css',
        'vendor/laravel-admin/toastr/build/toastr.min.css',
        'vendor/laravel-admin/bootstrap3-editable/css/bootstrap-editable.css',
        'vendor/laravel-admin/google-fonts/fonts.css',
        'vendor/laravel-admin/AdminLTE/dist/css/AdminLTE.min.css',
    ];

    /**
     * @var array
     */
    public static $baseJs = [
        'vendor/laravel-admin/AdminLTE/bootstrap/js/bootstrap.min.js',
        'vendor/laravel-admin/AdminLTE/plugins/slimScroll/jquery.slimscroll.min.js',
        'vendor/laravel-admin/AdminLTE/dist/js/app.min.js',
        'vendor/laravel-admin/jquery-pjax/jquery.pjax.js',
        'vendor/laravel-admin/nprogress/nprogress.js',
        'vendor/laravel-admin/nestable/jquery.nestable.js',
        'vendor/laravel-admin/toastr/build/toastr.min.js',
        'vendor/laravel-admin/bootstrap3-editable/js/bootstrap-editable.min.js',
        'vendor/laravel-admin/sweetalert2/dist/sweetalert2.min.js',
        'vendor/laravel-admin/laravel-admin/laravel-admin.js',
    ];

    /**
     * @var string
     */
    public static $jQuery = 'vendor/laravel-admin/AdminLTE/plugins/jQuery/jQuery-2.1.4.min.js';
```

代码实现

```php
// 等待替换的路径
$pathsToReplace = [
    'baseCss',
    'baseJs',
    'jQuery'
];

$assetBasePath = "https://static.tbxzs.com/libs/laravel-admin/1.8.1/"; // 存放静态资源的基础目录。
foreach ($pathsToReplace as $item) {
    // 取出存放目录的变量。
    $paths = &\Encore\Admin\Admin::$$item;

    // 批量替换主目录。
    if (is_array($paths)) {
        foreach ($paths as &$path) {
            $path = str_replace("vendor/laravel-admin/", $assetBasePath, $path);
        };
    } else {
        $path = str_replace("vendor/laravel-admin/", $assetBasePath, $path);
    }
}
```
替换后发现无法完全达到目的，如图。所以决定采用其他方案。
![](/media/15962545599789.jpg)
![](/media/15962613409578.jpg)

# 方案1：修改 admin_asset 方法。
`admin_asset` 函数是 Laravel Admin 自建的全局函数，用于处理静态文件的 http => https，所以可以安心覆盖修改。
该函数在 `src/helpers.php` 中被声明，该文件引用顺序排在绝大多数文件之前(如图 1、2)，且引用是写死在包声明中的，无法直接通过 `composer.json` 文件修改。
覆盖方案较多，目的都是为了在原方法之前声明函数：
- 修改 public/index.php 文件，将新函数卸载文件中。
- 通过 [composer-include-files](https://www.cnblogs.com/xdao/p/php_autoload_sort.html) 来优先引导外部函数，更优雅一些，但感觉没必要，因为本次改动较小。
- 使用 [override_function](https://www.php.net/manual/en/function.override-function.php) 、[runkit_function_redefine](https://www.php.net/manual/en/function.runkit-function-remove.php) 方法覆盖原函数，已放弃，因为需要安装 PHP 扩展。
- 延迟加载 Laravel Admin 的全局函数文件：`src/helpers.php`，相关文档：[mcaskill/composer-exclude-files](https://packagist.org/packages/mcaskill/composer-exclude-files)、[Exclude files from autoload_files.php](https://github.com/composer/composer/issues/5029)

暂定使用第 1 个方法，即修改 index.php 文件。因为非常较小且不折腾，若后续有较多需要修改的内容，可选用方案 2，更加通用且优雅。

![](/media/15962583269706.jpg)
![](/media/15962583881855.jpg)

方案 1 具体修改内容：
```
/*
 * 覆盖 Laravel Admin 的 admin_asset 方法，用于批量替换静态资源路径。
 *
 */

function admin_asset($path)
{
    $assetBasePath = "//static.tbxzs.com/libs/laravel-admin/1.8.1/"; // 存放静态资源的基础目录。
    $path          = preg_replace("/[\/]*vendor\/laravel-admin\//", $assetBasePath, $path); // 目录举例：vendor/laravel-admin/bootstrap3-editable/css/bootstrap-editable.css、/vendor/laravel-admin/bootstrap3-editable/css/bootstrap-editable.css

    return (config('admin.https') || config('admin.secure')) ? secure_asset($path) : asset($path);
}
```
![](/media/15962600910658.jpg)

## 效果
![](/media/15962617735540.jpg)


# 方案2：迁移到  Dcat-admin
Dcat-admin 的静态资源控制似乎更加灵活，待确认。