title: CI源码分析-index.php
date: 2016-02-12 21:17
tags: [CI,PHP]
categories: PHP
---


开始的开始。

<!-- more -->

# 知识点：

## 错误提示

    ini_set('display_errors', 1);

> 设置指定配置选项的值。这个选项会在脚本运行时保持新的值，并在脚本结束时恢复。

    int error_reporting ([ int $level ] )

> error_reporting() 函数能够在运行时设置 error_reporting 指令。 PHP 有诸多错误级别，使用该函数可以设置在脚本运行时的级别。 如果没有设置可选参数 level， error_reporting() 仅会返回当前的错误报告级别。 

这里有两种错误相关的函数：

    ini_set('display_errors', 0);
    error_reporting(E_ALL & ~E_NOTICE & ~E_STRICT & ~E_USER_NOTICE);

他们的区别是：

**display_errors** 是开启php错误提示，而且是**所有错误提示**， 开启错误提示，是为了方便调试修改，只有两种状态量：**On & Off**
**error_reporting** 可以选择性的关闭或者说忽略某些不想要的错误提示。

[关于 error_reporting 可选参数介绍](http://www.w3school.com.cn/php/func_error_reporting.asp)

## 文件/文件夹操作

    string realpath ( string $path ) 

> 获取 文件（或文件夹）绝对路径
> 
> realpath() 扩展所有的符号连接并且处理输入的 path 中的 '/./', '/../' 以及多余的 '/' 并返回规范化后的绝对路径名。返回的路径中没有符号连接，'/./' 或 '/../' 成分。 失败时返回 FALSE，比如说文件不存在的话。 


    string dirname ( string $path )

> 给出一个包含有指向一个文件的全路径的字符串，本函数返回去掉文件名后的目录名。

EG:

```php
	<?php
		echo "1) " . dirname("/etc/passwd") . PHP_EOL; // 1) /etc
		echo "2) " . dirname("/etc/") . PHP_EOL; // 2) / (or \ on Windows)
		echo "3) " . dirname("."); // 3) .
	?> 
```

    mixed pathinfo ( string $path [, int $options = PATHINFO_DIRNAME | PATHINFO_BASENAME | PATHINFO_EXTENSION | PATHINFO_FILENAME ] )

> `pathinfo()` 返回一个关联数组包含有 path 的信息。返回关联数组还是字符串取决于 options。

eg:

    print_r(pathinfo('/a/b/c.s'));

结果：

    Array ( [dirname] => /a/b [basename] => c.s [extension] => s [filename] => c )


## 字符串操作

    string strrchr ( string $haystack , mixed $needle )

> 该函数返回 haystack 字符串中的一部分，这部分以 needle 的最后出现位置开始，直到 haystack 末尾。

EG:

源码中利用此函数来获取 system folder的名字：

    define('SYSDIR', trim(strrchr(trim(BASEPATH, '/'), '/'), '/'));





---

    mixed version_compare ( string $version1 , string $version2 [, string $operator ] )


> 默认情况下，在第一个版本低于第二个时，version_compare() 返回 -1；如果两者相等，返回 0；第二个版本更低时则返回 1。 
> 
> 当使用了可选参数 operator 时，如果关系是操作符所指定的那个，函数将返回 TRUE，否则返回 FALSE。

---

    int error_reporting ([ int $level ] )

# 源码分析：

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
    
    /*
     *---------------------------------------------------------------
     * APPLICATION ENVIRONMENT
     *---------------------------------------------------------------
     *
     * You can load different configurations depending on your
     * current environment. Setting the environment also influences
     * things like logging and error reporting.
     *
     * This can be set to anything, but default usage is:
     *
     *     development
     *     testing
     *     production
     *
     * NOTE: If you change these, also change the error_reporting() code below
     */
    	define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'development');
    

> 定义了常量**ENVIRONMENT**，不同的级别显示不同的错误。

    /*
     *---------------------------------------------------------------
     * ERROR REPORTING
     *---------------------------------------------------------------
     *
     * Different environments will require different levels of error reporting.
     * By default development will show errors but testing and live will hide them.
     */
    switch (ENVIRONMENT)
    {
    	case 'development':
    		error_reporting(-1);
    		ini_set('display_errors', 1);
    	break;
    
    	case 'testing':
    	case 'production':
    		ini_set('display_errors', 0);
    		if (version_compare(PHP_VERSION, '5.3', '>='))
    		{
    			error_reporting(E_ALL & ~E_NOTICE & ~E_DEPRECATED & ~E_STRICT & ~E_USER_NOTICE & ~E_USER_DEPRECATED);
    		}
    		else
    		{
    			error_reporting(E_ALL & ~E_NOTICE & ~E_STRICT & ~E_USER_NOTICE);
    		}
    	break;
    
    	default:
    		header('HTTP/1.1 503 Service Unavailable.', TRUE, 503);
    		echo 'The application environment is not set correctly.';
    		exit(1); // EXIT_ERROR
    }
    
    /*
     *---------------------------------------------------------------
     * SYSTEM FOLDER NAME
     *---------------------------------------------------------------
     *
     * This variable must contain the name of your "system" folder.
     * Include the path if the folder is not in the same directory
     * as this file.
     */
    	$system_path = 'system';
    
    /*
     *---------------------------------------------------------------
     * APPLICATION FOLDER NAME
     *---------------------------------------------------------------
     *
     * If you want this front controller to use a different "application"
     * folder than the default one you can set its name here. The folder
     * can also be renamed or relocated anywhere on your server. If
     * you do, use a full server path. For more info please see the user guide:
     * http://codeigniter.com/user_guide/general/managing_apps.html
     *
     * NO TRAILING SLASH!
     */
    	$application_folder = 'application';
    
    /*
     *---------------------------------------------------------------
     * VIEW FOLDER NAME
     *---------------------------------------------------------------
     *
     * If you want to move the view folder out of the application
     * folder set the path to the folder here. The folder can be renamed
     * and relocated anywhere on your server. If blank, it will default
     * to the standard location inside your application folder. If you
     * do move this, use the full server path to this folder.
     *
     * NO TRAILING SLASH!
     */
    	$view_folder = '';
    
    
    /*
     * --------------------------------------------------------------------
     * DEFAULT CONTROLLER
     * --------------------------------------------------------------------
     *
     * Normally you will set your default controller in the routes.php file.
     * You can, however, force a custom routing by hard-coding a
     * specific controller class/function here. For most applications, you
     * WILL NOT set your routing here, but it's an option for those
     * special instances where you might want to override the standard
     * routing in a specific front controller that shares a common CI installation.
     *
     * IMPORTANT: If you set the routing here, NO OTHER controller will be
     * callable. In essence, this preference limits your application to ONE
     * specific controller. Leave the function name blank if you need
     * to call functions dynamically via the URI.
     *
     * Un-comment the $routing array below to use this feature
     */
    	// The directory name, relative to the "controllers" folder.  Leave blank
    	// if your controller is not in a sub-folder within the "controllers" folder
    	// $routing['directory'] = '';
    
    	// The controller class file name.  Example:  mycontroller
    	// $routing['controller'] = '';
    
    	// The controller function you wish to be called.
    	// $routing['function']	= '';
    
    
    /*
     * -------------------------------------------------------------------
     *  CUSTOM CONFIG VALUES
     * -------------------------------------------------------------------
     *
     * The $assign_to_config array below will be passed dynamically to the
     * config class when initialized. This allows you to set custom config
     * items or override any default config values found in the config.php file.
     * This can be handy as it permits you to share one application between
     * multiple front controller files, with each file containing different
     * config values.
     *
     * Un-comment the $assign_to_config array below to use this feature
     */
    	// $assign_to_config['name_of_config_item'] = 'value of config item';
    
    
    
    // --------------------------------------------------------------------
    // END OF USER CONFIGURABLE SETTINGS.  DO NOT EDIT BELOW THIS LINE
    // --------------------------------------------------------------------
    
    /*
     * ---------------------------------------------------------------
     *  Resolve the system path for increased reliability
     * ---------------------------------------------------------------
     */
    
    	// Set the current directory correctly for CLI requests
    	if (defined('STDIN'))
    	{
    		chdir(dirname(__FILE__));
    	}
    
    	if (($_temp = realpath($system_path)) !== FALSE)
    	{
    		$system_path = $_temp.'/';
    	}
    	else
    	{
    		// Ensure there's a trailing slash
    		$system_path = rtrim($system_path, '/').'/';
    	}
    
    	// Is the system path correct?
    	if ( ! is_dir($system_path))
    	{
    		header('HTTP/1.1 503 Service Unavailable.', TRUE, 503);
    		echo 'Your system folder path does not appear to be set correctly. Please open the following file and correct this: '.pathinfo(__FILE__, PATHINFO_BASENAME);
    		exit(3); // EXIT_CONFIG
    	}
    
    /*
     * -------------------------------------------------------------------
     *  Now that we know the path, set the main path constants
     * -------------------------------------------------------------------
     */
    	// The name of THIS file
    	define('SELF', pathinfo(__FILE__, PATHINFO_BASENAME));
    
    	// Path to the system folder
    	define('BASEPATH', str_replace('\\', '/', $system_path));
    
    	// Path to the front controller (this file)
    	define('FCPATH', dirname(__FILE__).'/');
    
    	// Name of the "system folder"
    	define('SYSDIR', trim(strrchr(trim(BASEPATH, '/'), '/'), '/'));
    
    	// The path to the "application" folder
    	if (is_dir($application_folder))
    	{
    		if (($_temp = realpath($application_folder)) !== FALSE)
    		{
    			$application_folder = $_temp;
    		}
    
    		define('APPPATH', $application_folder.DIRECTORY_SEPARATOR);
    	}
    	else
    	{
    		if ( ! is_dir(BASEPATH.$application_folder.DIRECTORY_SEPARATOR))
    		{
    			header('HTTP/1.1 503 Service Unavailable.', TRUE, 503);
    			echo 'Your application folder path does not appear to be set correctly. Please open the following file and correct this: '.SELF;
    			exit(3); // EXIT_CONFIG
    		}
    
    		define('APPPATH', BASEPATH.$application_folder.DIRECTORY_SEPARATOR);
    	}
    
    	// The path to the "views" folder
    	if ( ! is_dir($view_folder))
    	{
    		if ( ! empty($view_folder) && is_dir(APPPATH.$view_folder.DIRECTORY_SEPARATOR))
    		{
    			$view_folder = APPPATH.$view_folder;
    		}
    		elseif ( ! is_dir(APPPATH.'views'.DIRECTORY_SEPARATOR))
    		{
    			header('HTTP/1.1 503 Service Unavailable.', TRUE, 503);
    			echo 'Your view folder path does not appear to be set correctly. Please open the following file and correct this: '.SELF;
    			exit(3); // EXIT_CONFIG
    		}
    		else
    		{
    			$view_folder = APPPATH.'views';
    		}
    	}
    
    	if (($_temp = realpath($view_folder)) !== FALSE)
    	{
    		$view_folder = $_temp.DIRECTORY_SEPARATOR;
    	}
    	else
    	{
    		$view_folder = rtrim($view_folder, '/\\').DIRECTORY_SEPARATOR;
    	}
    
    	define('VIEWPATH', $view_folder);

>定义了一系列常量如application、view 的路径等。
>用到的函数：pathinfo、realpath、strrchr、dirname、is_dir、chdir、rtrim等
    
    /*
     * --------------------------------------------------------------------
     * LOAD THE BOOTSTRAP FILE
     * --------------------------------------------------------------------
     *
     * And away we go...
     */
    require_once BASEPATH.'core/CodeIgniter.php';




