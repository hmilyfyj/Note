---
title: 替换 Laravel-Admin 静态资源
date: 2020-08-01 12:18:35
tags: 折腾
categories: 折腾
---

# 原有替换方案

修改下列变量路径

```
\Encore\Admin\Admin::$baseJs
\Encore\Admin\Admin::$baseCss
\Encore\Admin\Admin::$jQuery
```

替换后发现无法完全达到目的，如图。所以决定采用其他方案。
![](/media/15962545599789.jpg)

# 方案1：修改 admin_asset 方法。
`admin_asset` 函数是 Laravel Admin 自建的全局函数，用于处理静态文件的 http => https，所以可以安心覆盖修改。函数所在的文件（helpers.php）的引用顺序排在绝大多数文件之前(如图 1、2)。覆盖方案较多，目的都是为了在原方法之前声明函数：
- 修改 public/index.php 文件，将新函数卸载文件中。
- 通过 [composer-include-files](https://www.cnblogs.com/xdao/p/php_autoload_sort.html) 来优先引导外部函数，更优雅一些，但感觉没必要，因为本次改动较小。
- 使用 [override_function](https://www.php.net/manual/en/function.override-function.php) 、[runkit_function_redefine](https://www.php.net/manual/en/function.runkit-function-remove.php) 方法覆盖原函数，已放弃，因为需要安装 PHP 扩展。
- 延迟加载 Laravel Admin 的全局函数文件：`src/helpers.php`，相关文档：[mcaskill/composer-exclude-files](https://packagist.org/packages/mcaskill/composer-exclude-files)、[Exclude files from autoload_files.php](https://github.com/composer/composer/issues/5029)

暂定使用第 1 个方法，即修改 index.php 文件。因为非常较小且不折腾，若后续有较多需要修改的内容，可选用方案 2，更加通用且优雅。

![](/media/15962583269706.jpg)
![](/media/15962583881855.jpg)

方案 1 具体修改内容：

![](/media/15962532699988.jpg)

# 方案2：迁移到  Dcat-admin
Dcat-admin 的静态资源控制似乎更加灵活，待确认。