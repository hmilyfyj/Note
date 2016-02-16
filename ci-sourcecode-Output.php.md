
title: CI源码分析-Output.php
date: 2016-02-16 16:49
tags: [CI,PHP]
categories: PHP

---
 CodeIgniter.php 引入的第七个核心文件（`$OUT`）。输出类是个核心类，它的功能只有一个：发送 Web 页面内容到请求的浏览器。 如果你开启缓存，它也负责 缓存 你的 Web 页面。

<!-- more -->

### zlib.output_compression

>对页面尽兴Gzip压缩

> 一般而言，页面文件开启gzip压缩以后，其体积可以减小60%~90%，对于文字类站点，可以节省下大量的带宽与用户等待时间。但是不论是iis还是apache默认都只压缩html类静态文件，对于php文件需要模块配置才可支持（iis7.5中开启动态+静态压缩也可以），于是利用php自身功能到达gzip的效果也成为一项合理的诉求。[参考地址](http://stackoverflow.com/questions/25363864/zlib-output-compression-and-output-buffering)

### Mime types

[参考连接](http://www.cnblogs.com/jsean/articles/1610265.html)

### 字符串操作

#### 字符串比较

strcmp strncmp strcasecmp strncasecmp

#### 字符串解析

    mixed sscanf ( string $str , string $format [, mixed &$... ] )

### 数组操作

    mixed current ( array &$array )

    array array_map ( callable $callback , array $arr1 [, array $... ] )

> array_map() 返回一个数组，该数组包含了 arr1 中的所有单元经过 callback 作用过之后的单元。callback
> 接受的参数数目应该和传递给 array_map() 函数的数组数目一致。

    mixed array_shift ( array &$array )

> array_shift() 将 array 的第一个单元移出并作为结果返回，将 array
> 的长度减一并将所有其它单元向前移动一位。所有的数字键名将改为从零开始计数，文字键名将不变。

    array headers_list ( void )

> headers_list() will return a list of headers to be sent to the browser
> / client. To determine whether or not these headers have been sent
> yet, use headers_sent().


    array array_merge ( array $array1 [, array $... ] )

> `array_merge()` 将一个或多个数组的单元合并起来，一个数组中的值附加在前一个数组的后面。返回作为结果的数组。 
> 
> 如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。然而，如果数组包含数字键名，后面的值将不会覆盖原来的值，而是附加到后面。
> 
> 
> 如果只给了一个数组并且该数组是数字索引的，则键名会以连续方式重新索引。

    bool ob_start ([ callback $output_callback [, int $chunk_size [, bool $erase ]]] )

> 此函数将打开输出缓冲。当输出缓冲激活后，脚本将不会输出内容（除http标头外），相反需要输出的内容被存储在内部缓冲区中。

[参考地址](http://www.jcwcn.com/article-16878-1.html)


    string ob_gzhandler ( string $buffer , int $mode )

> ob_gzhandler()目的是用在ob_start()中作回调函数，以方便将gz
> 编码的数据发送到支持压缩页面的浏览器。在ob_gzhandler()真正发送压缩过的数据之前，该
> 函数会确定（判定）浏览器可以接受哪种类型内容编码（"gzip","deflate",或者根本什么都不支持），然后 返回相应的输出。
> 所有可以发送正确头信息表明他自己可以接受压缩的网页的浏览器，都可以支持。 All browsers are supported since
> it's up to the browser to send the correct header saying that it
> accepts compressed web pages. 如果一个浏览器不支持压缩过的页面，此函数返回FALSE。

    bool flock ( resource $handle , int $operation [, int &$wouldblock ] )

> flock() 允许执行一个简单的可以在任何平台中使用的读取/写入模型（包括大部分的 Unix 派生版和甚至是 Windows）。

    array array_intersect_key ( array $array1 , array $array2 [, array $ ... ] )

> array_intersect_key() 返回一个数组，该数组包含了所有出现在 array1 中并同时出现在所有其它参数数组中的键名的值。 

    array array_flip ( array $trans )

> array_flip() 返回一个反转后的 array，例如 trans 中的键名变成了值，而 trans 中的值成了键名。 
> 
> 注意 trans 中的值需要能够作为合法的键名，例如需要是 integer 或者
> string。如果值的类型不对将发出一个警告，并且有问题的键／值对将不会反转。 
> 
> 如果同一个值出现了多次，则最后一个键名将作为它的值，所有其它的都丢失了。


 

``` PHP
```

