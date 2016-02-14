
title: CI源码分析-Benchmark.php
date: 2016-02-14 11:16
tags: [CI,PHP]
categories: PHP
description: "CodeIgniter.php 引入的第二个类(`$BM`)，用于基准测试。"
---


``` PHP
<?php
/**
 * CodeIgniter
 *
 * An open source application development framework for PHP
 *
 * This content is released under the MIT License (MIT)
 *
 * Copyright (c) 2014 - 2015, British Columbia Institute of Technology
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 *
 * @package	CodeIgniter
 * @author	EllisLab Dev Team
 * @copyright	Copyright (c) 2008 - 2014, EllisLab, Inc. (http://ellislab.com/)
 * @copyright	Copyright (c) 2014 - 2015, British Columbia Institute of Technology (http://bcit.ca/)
 * @license	http://opensource.org/licenses/MIT	MIT License
 * @link	http://codeigniter.com
 * @since	Version 1.0.0
 * @filesource
 */
defined('BASEPATH') OR exit('No direct script access allowed');

/**
 * Benchmark Class
 *
 * This class enables you to mark points and calculate the time difference
 * between them. Memory consumption can also be displayed.
 *
 * @package		CodeIgniter
 * @subpackage	Libraries
 * @category	Libraries
 * @author		EllisLab Dev Team
 * @link		http://codeigniter.com/user_guide/libraries/benchmark.html
 */
class CI_Benchmark {

	/**
	 * List of all benchmark markers
	 *
	 * @var	array
	 */
	public $marker = array();

	/**
	 * Set a benchmark marker
	 *
	 * Multiple calls to this function can be made so that several
	 * execution points can be timed.
	 * 记录时间点。
	 *
	 * @param	string	$name	Marker name
	 * @return	void
	 */
	public function mark($name)
	{
		$this->marker[$name] = microtime(TRUE);
	}
```

>函数：`mixed microtime ([ bool $get_as_float ] )` 本函数以 "msec sec" 的格式返回一个字符串，其中 sec 是自 Unix 纪元（0:00:00 January 1, 1970 GMT）起到现在的秒数，msec 是微秒部分。字符串的两部分都是以秒为单位返回的。 
>如果给出了 get_as_float 参数并且其值等价于 TRUE，microtime() 将返回一个浮点数。 

``` PHP
	// --------------------------------------------------------------------

	/**
	 * Elapsed time
	 *
	 * Calculates the time difference between two marked points.
	 *
	 * If the first parameter is empty this function instead returns the
	 * {elapsed_time} pseudo-variable. This permits the full system
	 * execution time to be shown in a template. The output class will
	 * swap the real value for this variable.
	 * 计算两点间的消耗时间
	 *
	 * @param	string	$point1		A particular marked point
	 * @param	string	$point2		A particular marked point
	 * @param	int	$decimals	Number of decimal places
	 *
	 * @return	string	Calculated elapsed time on success,
	 *			an '{elapsed_string}' if $point1 is empty
	 *			or an empty string if $point1 is not found.
	 */
	public function elapsed_time($point1 = '', $point2 = '', $decimals = 4)
	{
		if ($point1 === '')
		{
			return '{elapsed_time}';
		}

		if ( ! isset($this->marker[$point1]))
		{
			return '';
		}

		if ( ! isset($this->marker[$point2]))
		{
			$this->marker[$point2] = microtime(TRUE);
		}

		return number_format($this->marker[$point2] - $this->marker[$point1], $decimals);
	}
```

>函数:`string number_format ( float $number [, int $decimals = 0 ] )`
>本函数可以接受1个、2个或者4个参数（注意：不能是3个）: 
> 
> 如果只提供第一个参数，number的小数部分会被去掉 并且每个千位分隔符都是英文小写逗号"," 
> 
> 如果提供两个参数，number将保留小数点后的位数到你设定的值，其余同楼上 
> 
> 如果提供了四个参数，number 将保留decimals个长度的小数部分,
> 小数点被替换为dec_point，千位分隔符替换为thousands_sep


``` PHP
	// --------------------------------------------------------------------

	/**
	 * Memory Usage
	 *
	 * Simply returns the {memory_usage} marker.
	 *
	 * This permits it to be put it anywhere in a template
	 * without the memory being calculated until the end.
	 * The output class will swap the real value for this variable.
	 *
	 * @return	string	'{memory_usage}'
	 */
	public function memory_usage()
	{
		return '{memory_usage}';
	}

}

```

