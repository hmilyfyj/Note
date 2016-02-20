title: CI源码分析-CodeIgniter.php
date: 2016-02-13 10:24
tags: [CI,PHP]
categories: CI
---

这里把各个类库串联了起来。

<!-- more -->

# 知识点


`$GLOBALS`  一个包含了全部变量的全局组合数组。变量的名字就是数组的键。 


# 流程

1. 引入 Constants.php 定义全局变量。
2. 引入 Common.php 定义全局函数。
3. 当 php version 小于 5.4 时，处理 register_globals 带来的安全问题。
4. 绑定 error handler
4.1 set_error_handler('_error_handler');
4.2 set_exception_handler('_exception_handler');
4.3 register_shutdown_function('_shutdown_handler');
5. 修改自定义类前缀`subclass_prefix`（默认MY_）
6. 定义 comoser 的 autoloader
7.  引入 Benchmark 类（`$BM`）
8. 引入 Hooks 类 （`$EXT`）
9. 执行钩子 pre_system
10. 引入 Config 类（`$CFG`）
11. 设置编码 
11.1 mbstring.internal_encoding
11.2 iconv.internal_encoding
11.3 php.internal_encoding
12. 引入 BASEPATH.'core/compat/*' 相关函数
12.1 mbstring.php
12.2 hash.php
12.3 password.php
12.4 standard.php
13. 引入核心类
13.1  Utf-8 类 （`$UNI`）
13.2  Uri 类（`$URI`）
13.3 Router（`$RTR`）尽兴路由检测
13.4  Outut （`$OUT`）
14. 检测 cache_override 参数，如果不存在则检测是否有 缓存 存在。
15.  继续引入类
15.1 Input 类（`$IN`）
15.2 Lang 类（`$LANG`）
15.3 Controller 类
15.3.4 引入重写的 Controller 类
16. 404 检测
17. 检测 _remap 函数调用参数
18. 调用钩子 pre_controller
19. 创建请求的类 $CI 
20. 执行钩子 post_controller_constructor
21. 执行请求的方法
22. 执行钩子 post_controller
23. 执行钩子 display_override ，如果没有，则正常显示
24. 执行钩子 post_system

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
     * System Initialization File
     *
     * Loads the base classes and executes the request.
     *
     * @package		CodeIgniter
     * @subpackage	CodeIgniter
     * @category	Front-controller
     * @author		EllisLab Dev Team
     * @link		http://codeigniter.com/user_guide/
     */
    
    /**
     * CodeIgniter Version
     *
     * @var	string
     *
     */
    	define('CI_VERSION', '3.0.2');
    
    /*
     * ------------------------------------------------------
     *  Load the framework constants
     * ------------------------------------------------------
     */
    	if (file_exists(APPPATH.'config/'.ENVIRONMENT.'/constants.php'))
    	{
    		require_once(APPPATH.'config/'.ENVIRONMENT.'/constants.php');
    	}
    
    	require_once(APPPATH.'config/constants.php');
    
    /*
     * ------------------------------------------------------
     *  Load the global functions
     * ------------------------------------------------------
     */
    	require_once(BASEPATH.'core/Common.php');
    
    
  >定义CI_VERSION，引入constants.php、Common.php

>`ini_set('magic_quotes_runtime', 0);`
>
> magic_quotes_runtime
> 
> 作用：对通过 fread()、file_get_contents() 返回的文本执行 addslashes() 操作，对执行sql查询的结果执行 addslashes() 操作。 作用范围：从文件中读取的数据或执行 exec() 的结果或是从SQL查询中得到的。 作用时间：每次当脚本访问运行状态中产生的数据。
>[参考地址](http://www.cnblogs.com/adforce/p/3531151.html)
>[参考地址2](http://www.phpddt.com/php/php-magic-quotes.html)


    /*
     * ------------------------------------------------------
     * Security procedures
     * ------------------------------------------------------
     */
    
    // 小于 5.4 版本存在 register_globals 参数引起的安全问题
    if ( ! is_php('5.4'))
    {
    	ini_set('magic_quotes_runtime', 0);
    
    	if ((bool) ini_get('register_globals'))
    	{
    		$_protected = array(
    			'_SERVER',
    			'_GET',
    			'_POST',
    			'_FILES',
    			'_REQUEST',
    			'_SESSION',
    			'_ENV',
    			'_COOKIE',
    			'GLOBALS',
    			'HTTP_RAW_POST_DATA',
    			'system_path',
    			'application_folder',
    			'view_folder',
    			'_protected',
    			'_registered'
    		);
    
    		$_registered = ini_get('variables_order');
    		foreach (array('E' => '_ENV', 'G' => '_GET', 'P' => '_POST', 'C' => '_COOKIE', 'S' => '_SERVER') as $key => $superglobal)
    		{
    			if (strpos($_registered, $key) === FALSE)
    			{
    				continue;
    			}
    
    			foreach (array_keys($$superglobal) as $var)
    			{
    				if (isset($GLOBALS[$var]) && ! in_array($var, $_protected, TRUE))
    				{
    					$GLOBALS[$var] = NULL;
    				}
    			}
    		}
    	}
    }

>这一段是为了处理register_globals参数开启带来的安全问题。
[register_globals](http://stackoverflow.com/questions/3593210/what-are-register-globals-in-php)

[what-does-egpcs-mean-in-php](http://stackoverflow.com/questions/1312871/what-does-egpcs-mean-in-php)
    
    
    /*
     * ------------------------------------------------------
     *  Define a custom error handler so we can log PHP errors
     * ------------------------------------------------------
     */
    	set_error_handler('_error_handler');
    	set_exception_handler('_exception_handler');
    	register_shutdown_function('_shutdown_handler');
  
  >异常处理
  
  
    /*
     * ------------------------------------------------------
     *  Set the subclass_prefix
     * ------------------------------------------------------
     *
     * Normally the "subclass_prefix" is set in the config file.
     * The subclass prefix allows CI to know if a core class is
     * being extended via a library in the local application
     * "libraries" folder. Since CI allows config items to be
     * overridden via data set in the main index.php file,
     * before proceeding we need to know if a subclass_prefix
     * override exists. If so, we will set this value now,
     * before any classes are loaded
     * Note: Since the config file data is cached it doesn't
     * hurt to load it here.
     */
    	if ( ! empty($assign_to_config['subclass_prefix']))
    	{
    		get_config(array('subclass_prefix' => $assign_to_config['subclass_prefix']));
    	}
    
    /*
     * ------------------------------------------------------
     *  Should we use a Composer autoloader?
     * ------------------------------------------------------
     */
    	if ($composer_autoload = config_item('composer_autoload'))
    	{
    		if ($composer_autoload === TRUE)
    		{
    			file_exists(APPPATH.'vendor/autoload.php')
    				? require_once(APPPATH.'vendor/autoload.php')
    				: log_message('error', '$config[\'composer_autoload\'] is set to TRUE but '.APPPATH.'vendor/autoload.php was not found.');
    		}
    		elseif (file_exists($composer_autoload))
    		{
    			require_once($composer_autoload);
    		}
    		else
    		{
    			log_message('error', 'Could not find the specified $config[\'composer_autoload\'] path: '.$composer_autoload);
    		}
    	}
    
    /*
     * ------------------------------------------------------
     *  Start the timer... tick tock tick tock...
     * ------------------------------------------------------
     */
    	$BM =& load_class('Benchmark', 'core');
    	$BM->mark('total_execution_time_start');
    	$BM->mark('loading_time:_base_classes_start');
    
    /*
     * ------------------------------------------------------
     *  Instantiate the hooks class
     * ------------------------------------------------------
     */
    	$EXT =& load_class('Hooks', 'core');
    
    /*
     * ------------------------------------------------------
     *  Is there a "pre_system" hook?
     * ------------------------------------------------------
     */
    	$EXT->call_hook('pre_system');
    
    /*
     * ------------------------------------------------------
     *  Instantiate the config class
     * ------------------------------------------------------
     *
     * Note: It is important that Config is loaded first as
     * most other classes depend on it either directly or by
     * depending on another class that uses it.
     *
     */
    	$CFG =& load_class('Config', 'core');
    
    	// Do we have any manually set config items in the index.php file?
    	if (isset($assign_to_config) && is_array($assign_to_config))
    	{
    		foreach ($assign_to_config as $key => $value)
    		{
    			$CFG->set_item($key, $value);
    		}
    	}
    
  >Composer 的配置
  >引入核心类：Benchmark、Hooks（执行 pre_system hooks）、Config、
  
    /*
     * ------------------------------------------------------
     * Important charset-related stuff
     * ------------------------------------------------------
     *
     * Configure mbstring and/or iconv if they are enabled
     * and set MB_ENABLED and ICONV_ENABLED constants, so
     * that we don't repeatedly do extension_loaded() or
     * function_exists() calls.
     *
     * Note: UTF-8 class depends on this. It used to be done
     * in it's constructor, but it's _not_ class-specific.
     *
     */
    	$charset = strtoupper(config_item('charset'));
    	ini_set('default_charset', $charset);
    
    	if (extension_loaded('mbstring'))
    	{
    		define('MB_ENABLED', TRUE);
    		// mbstring.internal_encoding is deprecated starting with PHP 5.6
    		// and it's usage triggers E_DEPRECATED messages.
    		@ini_set('mbstring.internal_encoding', $charset);
    		// This is required for mb_convert_encoding() to strip invalid characters.
    		// That's utilized by CI_Utf8, but it's also done for consistency with iconv.
    		mb_substitute_character('none');
    	}
    	else
    	{
    		define('MB_ENABLED', FALSE);
    	}
    
    	// There's an ICONV_IMPL constant, but the PHP manual says that using
    	// iconv's predefined constants is "strongly discouraged".
    	if (extension_loaded('iconv'))
    	{
    		define('ICONV_ENABLED', TRUE);
    		// iconv.internal_encoding is deprecated starting with PHP 5.6
    		// and it's usage triggers E_DEPRECATED messages.
    		@ini_set('iconv.internal_encoding', $charset);
    	}
    	else
    	{
    		define('ICONV_ENABLED', FALSE);
    	}
    
    	if (is_php('5.6'))
    	{
    		ini_set('php.internal_encoding', $charset);
    	}
    
>配置charest以及mbstring扩展
>[官方文档](http://php.net/manual/zh/intro.mbstring.php)


    /*
     * ------------------------------------------------------
     *  Load compatibility features
     * ------------------------------------------------------
     */
    
    	require_once(BASEPATH.'core/compat/mbstring.php');
    	require_once(BASEPATH.'core/compat/hash.php');
    	require_once(BASEPATH.'core/compat/password.php');
    	require_once(BASEPATH.'core/compat/standard.php');
    
    /*
     * ------------------------------------------------------
     *  Instantiate the UTF-8 class
     * ------------------------------------------------------
     */
    	$UNI =& load_class('Utf8', 'core');
    
    /*
     * ------------------------------------------------------
     *  Instantiate the URI class
     * ------------------------------------------------------
     */
    	$URI =& load_class('URI', 'core');
    
    /*
     * ------------------------------------------------------
     *  Instantiate the routing class and set the routing
     * ------------------------------------------------------
     */
    	$RTR =& load_class('Router', 'core', isset($routing) ? $routing : NULL);
    
    /*
     * ------------------------------------------------------
     *  Instantiate the output class
     * ------------------------------------------------------
     */
    	$OUT =& load_class('Output', 'core');
    
    /*
     * ------------------------------------------------------
     *	Is there a valid cache file? If so, we're done...
     * ------------------------------------------------------
     */
    	if ($EXT->call_hook('cache_override') === FALSE && $OUT->_display_cache($CFG, $URI) === TRUE)
    	{
    		exit;
    	}
    
 >引入 compat下的内容：【待完善】。
 >引入类Utf8、Router、Output
 >检测是否需要输出缓存
 >
 >
    /*
     * -----------------------------------------------------
     * Load the security class for xss and csrf support
     * -----------------------------------------------------
     */
    	$SEC =& load_class('Security', 'core');
    
    /*
     * ------------------------------------------------------
     *  Load the Input class and sanitize globals
     * ------------------------------------------------------
     */
    	$IN	=& load_class('Input', 'core');
    
    /*
     * ------------------------------------------------------
     *  Load the Language class
     * ------------------------------------------------------
     */
    	$LANG =& load_class('Lang', 'core');
    
    /*
     * ------------------------------------------------------
     *  Load the app controller and local controller
     * ------------------------------------------------------
     *
     */
    	// Load the base controller class
    	require_once BASEPATH.'core/Controller.php';
    
    	/**
    	 * Reference to the CI_Controller method.
    	 *
    	 * Returns current CI instance object
    	 *
    	 * @return object
    	 */
    	function &get_instance()
    	{
    		return CI_Controller::get_instance();
    	}
    
    	if (file_exists(APPPATH.'core/'.$CFG->config['subclass_prefix'].'Controller.php'))
    	{
    		require_once APPPATH.'core/'.$CFG->config['subclass_prefix'].'Controller.php';
    	}
    
    	// Set a mark point for benchmarking
    	$BM->mark('loading_time:_base_classes_end');
    
    /*
     * ------------------------------------------------------
     *  Sanity checks
     * ------------------------------------------------------
     *
     *  The Router class has already validated the request,
     *  leaving us with 3 options here:
     *
     *	1) an empty class name, if we reached the default
     *	   controller, but it didn't exist;
     *	2) a query string which doesn't go through a
     *	   file_exists() check
     *	3) a regular request for a non-existing page
     *
     *  We handle all of these as a 404 error.
     *
     *  Furthermore, none of the methods in the app controller
     *  or the loader class can be called via the URI, nor can
     *  controller methods that begin with an underscore.
     */
    
    	$e404 = FALSE;
    	$class = ucfirst($RTR->class);
    	$method = $RTR->method;
    
    	if (empty($class) OR ! file_exists(APPPATH.'controllers/'.$RTR->directory.$class.'.php'))
    	{
    		$e404 = TRUE;
    	}
    	else
    	{
    		require_once(APPPATH.'controllers/'.$RTR->directory.$class.'.php');
    
    		if ( ! class_exists($class, FALSE) OR $method[0] === '_' OR method_exists('CI_Controller', $method))
    		{
    			$e404 = TRUE;
    		}
    		elseif (method_exists($class, '_remap'))
    		{
    			$params = array($method, array_slice($URI->rsegments, 2));
    			$method = '_remap';
    		}
    		// WARNING: It appears that there are issues with is_callable() even in PHP 5.2!
    		// Furthermore, there are bug reports and feature/change requests related to it
    		// that make it unreliable to use in this context. Please, DO NOT change this
    		// work-around until a better alternative is available.
    		elseif ( ! in_array(strtolower($method), array_map('strtolower', get_class_methods($class)), TRUE))
    		{
    			$e404 = TRUE;
    		}
    	}
    
    	if ($e404)
    	{
    		if ( ! empty($RTR->routes['404_override']))
    		{
    			if (sscanf($RTR->routes['404_override'], '%[^/]/%s', $error_class, $error_method) !== 2)
    			{
    				$error_method = 'index';
    			}
    
    			$error_class = ucfirst($error_class);
    
    			if ( ! class_exists($error_class, FALSE))
    			{
    				if (file_exists(APPPATH.'controllers/'.$RTR->directory.$error_class.'.php'))
    				{
    					require_once(APPPATH.'controllers/'.$RTR->directory.$error_class.'.php');
    					$e404 = ! class_exists($error_class, FALSE);
    				}
    				// Were we in a directory? If so, check for a global override
    				elseif ( ! empty($RTR->directory) && file_exists(APPPATH.'controllers/'.$error_class.'.php'))
    				{
    					require_once(APPPATH.'controllers/'.$error_class.'.php');
    					if (($e404 = ! class_exists($error_class, FALSE)) === FALSE)
    					{
    						$RTR->directory = '';
    					}
    				}
    			}
    			else
    			{
    				$e404 = FALSE;
    			}
    		}
    
    		// Did we reset the $e404 flag? If so, set the rsegments, starting from index 1
    		if ( ! $e404)
    		{
    			$class = $error_class;
    			$method = $error_method;
    
    			$URI->rsegments = array(
    				1 => $class,
    				2 => $method
    			);
    		}
    		else
    		{
    			show_404($RTR->directory.$class.'/'.$method);
    		}
    	}
    
    	if ($method !== '_remap')
    	{
    		$params = array_slice($URI->rsegments, 2);
    	}
    
    /*
     * ------------------------------------------------------
     *  Is there a "pre_controller" hook?
     * ------------------------------------------------------
     */
    	$EXT->call_hook('pre_controller');
    
    /*
     * ------------------------------------------------------
     *  Instantiate the requested controller
     * ------------------------------------------------------
     */
    	// Mark a start point so we can benchmark the controller
    	$BM->mark('controller_execution_time_( '.$class.' / '.$method.' )_start');
    
    	$CI = new $class();
    
    /*
     * ------------------------------------------------------
     *  Is there a "post_controller_constructor" hook?
     * ------------------------------------------------------
     */
    	$EXT->call_hook('post_controller_constructor');
    
    /*
     * ------------------------------------------------------
     *  Call the requested method
     * ------------------------------------------------------
     */
    	call_user_func_array(array(&$CI, $method), $params);
    
    	// Mark a benchmark end point
    	$BM->mark('controller_execution_time_( '.$class.' / '.$method.' )_end');
    
    /*
     * ------------------------------------------------------
     *  Is there a "post_controller" hook?
     * ------------------------------------------------------
     */
    	$EXT->call_hook('post_controller');
    
    /*
     * ------------------------------------------------------
     *  Send the final rendered output to the browser
     * ------------------------------------------------------
     */
    	if ($EXT->call_hook('display_override') === FALSE)
    	{
    		$OUT->_display();
    	}
    
    /*
     * ------------------------------------------------------
     *  Is there a "post_system" hook?
     * ------------------------------------------------------
     */
    	$EXT->call_hook('post_system');


