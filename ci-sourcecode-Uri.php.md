
title: CI源码分析-Uri.php
date: 2016-02-15 16:06
tags: [CI,PHP]
categories: PHP

---
 CodeIgniter.php 引入的第五个核心文件（`$URI`）。URI 类用于帮助你从 URI 字符串中获取信息，如果你使用 URI 路由， 你也可以从路由后的 URI 中获取信息。 

<!-- more -->


>等待消化的知识：PHP预定义常量、PHP运行方式(SAPI)

>tips:string 中的字符可以通过一个从 0 开始的下标，用类似 array 结构中的方括号包含对应的数字来访问和修改，比如 $str[42]。可以把 string 当成字符组成的 array。函数 substr() 和 substr_replace() 可用于操作多于一个字符的情况。 这是我一直以来没注意的，所以说基础要打好。

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
 * URI Class
 *
 * Parses URIs and determines routing
 *
 * @package		CodeIgniter
 * @subpackage	Libraries
 * @category	URI
 * @author		EllisLab Dev Team
 * @link		http://codeigniter.com/user_guide/libraries/uri.html
 */
class CI_URI {

	/**
	 * List of cached URI segments
	 *
	 * @var	array
	 */
	public $keyval = array();

	/**
	 * Current URI string
	 *
	 * @var	string
	 */
	public $uri_string = '';

	/**
	 * List of URI segments
	 *
	 * Starts at 1 instead of 0.
	 *
	 * @var	array
	 */
	public $segments = array();

	/**
	 * List of routed URI segments
	 *
	 * Starts at 1 instead of 0.
	 *
	 * @var	array
	 */
	public $rsegments = array();

	/**
	 * Permitted URI chars
	 *
	 * PCRE character group allowed in URI segments
	 *
	 * @var	string
	 */
	protected $_permitted_uri_chars;

	/**
	 * Class constructor
	 *
	 * @return	void
	 */
	public function __construct()
	{
		$this->config =& load_class('Config', 'core');

		// If query strings are enabled, we don't need to parse any segments.
		// However, they don't make sense under CLI.
		if (is_cli() OR $this->config->item('enable_query_strings') !== TRUE)
		{
			$this->_permitted_uri_chars = $this->config->item('permitted_uri_chars');

			// If it's a CLI request, ignore the configuration
			if (is_cli())
			{
				$uri = $this->_parse_argv();
			}
			else
			{
				$protocol = $this->config->item('uri_protocol');
				empty($protocol) && $protocol = 'REQUEST_URI';

				switch ($protocol)
				{
					case 'AUTO': // For BC purposes only
					case 'REQUEST_URI':
						$uri = $this->_parse_request_uri();
						break;
					case 'QUERY_STRING':
						$uri = $this->_parse_query_string();
						break;
					case 'PATH_INFO':
					default:
						$uri = isset($_SERVER[$protocol])
							? $_SERVER[$protocol]
							: $this->_parse_request_uri();
						break;
				}
			}

			$this->_set_uri_string($uri);
		}

		log_message('info', 'URI Class Initialized');
	}
```

> `uri_protocol`可选项有`AUTO`、`PATH_INFO`、`QUERY_STRING`、`REQUEST_URI`、`ORIG_PATH_INFO`
> 含义分别如下： 
> `QUERY_STRING`:查询字符串；
> `PATH_INFO`：客户端提供的路径信息，即在实际执行脚本后面尾随的内容，会去掉Query String；
> `REQUEST_URI`：包含HTTP协议中定义的URI内容。
> 访问：http://pc.local/index.php/product/pc/summary?a=1时
> `PATH_INFO`为/product/pc/summary；`REQUEST_URI`为/index.php/product/pc/summary?a=1；`QUERY_STRING`为a=1

``` PHP
	// --------------------------------------------------------------------

	/**
	 * Set URI String
	 * 初始化 segments 数组。uri各段按照从左往右的顺序一次放入数组中。
	 *
	 * @param 	string	$str
	 * @return	void
	 */
	protected function _set_uri_string($str)
	{
		// Filter out control characters and trim slashes
		$this->uri_string = trim(remove_invisible_characters($str, FALSE), '/');

		if ($this->uri_string !== '')
		{
			// Remove the URL suffix, if present
			if (($suffix = (string) $this->config->item('url_suffix')) !== '')
			{
				$slen = strlen($suffix);

				if (substr($this->uri_string, -$slen) === $suffix)
				{
					$this->uri_string = substr($this->uri_string, 0, -$slen);
				}
			}

			$this->segments[0] = NULL;
			// Populate the segments array
			foreach (explode('/', trim($this->uri_string, '/')) as $val)
			{
				$val = trim($val);
				// Filter segments for security
				$this->filter_uri($val);

				if ($val !== '')
				{
					$this->segments[] = $val;
				}
			}
			
			//移除占位字段
			unset($this->segments[0]);
		}
	}

	// --------------------------------------------------------------------

	/**
	 * Parse REQUEST_URI
	 *
	 * Will parse REQUEST_URI and automatically detect the URI from it,
	 * while fixing the query string if necessary.
	 *
	 * @return	string
	 */
	protected function _parse_request_uri()
	{
		
		//$url=http://www.qing.ga/welcome/tt?s=1
		//$_SERVER['REQUEST_URI']=/welcome/tt?s=1
		//$SCRIPT_NAME=/index.php
		if ( ! isset($_SERVER['REQUEST_URI'], $_SERVER['SCRIPT_NAME']))
		{
			return '';
		}

		// parse_url() returns false if no host is present, but the path or query string
		// contains a colon followed by a number
		$uri = parse_url('http://dummy'.$_SERVER['REQUEST_URI']);
		$query = isset($uri['query']) ? $uri['query'] : '';
		$uri = isset($uri['path']) ? $uri['path'] : '';

		//字符串可以用数组的形式来操作
		if (isset($_SERVER['SCRIPT_NAME'][0]))
		{
			if (strpos($uri, $_SERVER['SCRIPT_NAME']) === 0)
			{
				$uri = (string) substr($uri, strlen($_SERVER['SCRIPT_NAME']));
			}
			elseif (strpos($uri, dirname($_SERVER['SCRIPT_NAME'])) === 0)
			{
				$uri = (string) substr($uri, strlen(dirname($_SERVER['SCRIPT_NAME'])));
			}
		}

		// This section ensures that even on servers that require the URI to be in the query string (Nginx) a correct
		// URI is found, and also fixes the QUERY_STRING server var and $_GET array.
		if (trim($uri, '/') === '' && strncmp($query, '/', 1) === 0)
		{
			$query = explode('?', $query, 2);
			$uri = $query[0];
			$_SERVER['QUERY_STRING'] = isset($query[1]) ? $query[1] : '';
		}
		else
		{
			$_SERVER['QUERY_STRING'] = $query;
		}

		//讲处理后的请求参数放入$_GET数组
		parse_str($_SERVER['QUERY_STRING'], $_GET);

		if ($uri === '/' OR $uri === '')
		{
			return '/';
		}

		// Do some final cleaning of the URI and return it
		// 移除 ../、 ////等无意义非法字符
		return $this->_remove_relative_directory($uri);
	}
```	

> 	`void parse_str ( string $str [, array &$arr ] )` 如果 str 是 URL
> 传递入的查询字符串（query string），则将它解析为变量并设置到当前作用域。  
> 	拓展：数组合并 `+`、  `array_unshift`、`arrar_merge` 	
> `int strncmp ( string $str1 , string $str2 , int $len )` 该函数与 `strcmp()` 类似，不同之处在于你可以指定两个字符串比较时使用的长度（即最大比较长度）。

		


``` PHP
	// --------------------------------------------------------------------

	/**
	 * Parse QUERY_STRING
	 *
	 * Will parse QUERY_STRING and automatically detect the URI from it.
	 *
	 * @return	string
	 */
	protected function _parse_query_string()
	{
		$uri = isset($_SERVER['QUERY_STRING']) ? $_SERVER['QUERY_STRING'] : @getenv('QUERY_STRING');

		if (trim($uri, '/') === '')
		{
			return '';
		}
		elseif (strncmp($uri, '/', 1) === 0)
		{
			$uri = explode('?', $uri, 2);
			$_SERVER['QUERY_STRING'] = isset($uri[1]) ? $uri[1] : '';
			$uri = $uri[0];
		}

		parse_str($_SERVER['QUERY_STRING'], $_GET);

		return $this->_remove_relative_directory($uri);
	}

	// --------------------------------------------------------------------

	/**
	 * Parse CLI arguments
	 * 获得命令行传递的参数：0号参数为文件名
	 *
	 * Take each command line argument and assume it is a URI segment.
	 *
	 * @return	string
	 */
	protected function _parse_argv()
	{
		//取出参数
		$args = array_slice($_SERVER['argv'], 1);
		
		//返回数组形的参数，如果参数不存在则返回空
		return $args ? implode('/', $args) : '';
	}

	// --------------------------------------------------------------------

	/**
	 * Remove relative directory (../) and multi slashes (///)
	 * 安全：移除 ../、///
	 * 
	 * Do some final cleaning of the URI and return it, currently only used in self::_parse_request_uri()
	 *
	 * @param	string	$url
	 * @return	string
	 */
	protected function _remove_relative_directory($uri)
	{
		$uris = array();
		$tok = strtok($uri, '/');
		while ($tok !== FALSE)
		{
			if (( ! empty($tok) OR $tok === '0') && $tok !== '..')
			{
				$uris[] = $tok;
			}
			$tok = strtok('/');
		}

		return implode('/', $uris);
	}
```

> 	`string strtok ( string $str , string $token )`:strtok() 将字符串 str 分割为若干子字符串，每个子字符串以 token 中的字符分割。这也就意味着，如果有个字符串是 "This is an example  string"，你可以使用空格字符将这句话分割成独立的单词。  	注意仅第一次调用 strtok 函数时使用 string  参数。后来每次调用 strtok，都将只使用 token 参数，因为它会记住它在字符串 string 中的位置。如果要重新开始分割一个新的字符串，你需要再次使用 string 来调用 strtok 函数，以便完成初始化工作。注意可以在 token 参数中使用多个字符。字符串将被该参数中任何一个字符分割。
``` PHP
	// --------------------------------------------------------------------

	/**
	 * Filter URI
	 *
	 * Filters segments for malicious characters.
	 * 过滤字符。
	 *
	 * @param	string	$str
	 * @return	void
	 */
	public function filter_uri(&$str)
	{
		if ( ! empty($str) && ! empty($this->_permitted_uri_chars) && ! preg_match('/^['.$this->_permitted_uri_chars.']+$/i'.(UTF8_ENABLED ? 'u' : ''), $str))
		{
			show_error('The URI you submitted has disallowed characters.', 400);
		}
	}

	// --------------------------------------------------------------------

	/**
	 * Fetch URI Segment
	 *
	 * @see		CI_URI::$segments
	 * @param	int		$n		Index
	 * @param	mixed		$no_result	What to return if the segment index is not found
	 * @return	mixed
	 */
	public function segment($n, $no_result = NULL)
	{
		return isset($this->segments[$n]) ? $this->segments[$n] : $no_result;
	}

	// --------------------------------------------------------------------

	/**
	 * Fetch URI "routed" Segment
	 *
	 * Returns the re-routed URI segment (assuming routing rules are used)
	 * based on the index provided. If there is no routing, will return
	 * the same result as CI_URI::segment().
	 *
	 * @see		CI_URI::$rsegments
	 * @see		CI_URI::segment()
	 * @param	int		$n		Index
	 * @param	mixed		$no_result	What to return if the segment index is not found
	 * @return	mixed
	 */
	public function rsegment($n, $no_result = NULL)
	{
		return isset($this->rsegments[$n]) ? $this->rsegments[$n] : $no_result;
	}

	// --------------------------------------------------------------------

	/**
	 * URI to assoc
	 *
	 * Generates an associative array of URI data starting at the supplied
	 * segment index. For example, if this is your URI:
	 *
	 *	example.com/user/search/name/joe/location/UK/gender/male
	 *
	 * You can use this method to generate an array with this prototype:
	 *
	 *	array (
	 *		name => joe
	 *		location => UK
	 *		gender => male
	 *	 )
	 *
	 * @param	int	$n		Index (default: 3)
	 * @param	array	$default	Default values
	 * @return	array
	 */
	public function uri_to_assoc($n = 3, $default = array())
	{
		return $this->_uri_to_assoc($n, $default, 'segment');
	}

	// --------------------------------------------------------------------

	/**
	 * Routed URI to assoc
	 *
	 * Identical to CI_URI::uri_to_assoc(), only it uses the re-routed
	 * segment array.
	 *
	 * @see		CI_URI::uri_to_assoc()
	 * @param 	int	$n		Index (default: 3)
	 * @param 	array	$default	Default values 用于设置默认的键名，这样即使 URI 中缺少某个键名，也能保证返回的数组中包含该索引。
	 * @return 	array
	 */
	public function ruri_to_assoc($n = 3, $default = array())
	{
		return $this->_uri_to_assoc($n, $default, 'rsegment');
	}

	// --------------------------------------------------------------------

	/**
	 * Internal URI-to-assoc
	 *
	 * Generates a key/value pair from the URI string or re-routed URI string.
	 *
	 * @used-by	CI_URI::uri_to_assoc()
	 * @used-by	CI_URI::ruri_to_assoc()
	 * @param	int	$n		Index (default: 3)
	 * @param	array	$default	Default values
	 * @param	string	$which		Array name ('segment' or 'rsegment')
	 * @return	array
	 */
	protected function _uri_to_assoc($n = 3, $default = array(), $which = 'segment')
	{
		if ( ! is_numeric($n))
		{
			return $default;
		}

		//缓存
		if (isset($this->keyval[$which], $this->keyval[$which][$n]))
		{
			return $this->keyval[$which][$n];
		}

		$total_segments = "total_{$which}s";
		$segment_array = "{$which}_array";

		if ($this->$total_segments() < $n)
		{
			return (count($default) === 0)
				? array()
				: array_fill_keys($default, NULL);
		}

		$segments = array_slice($this->$segment_array(), ($n - 1));
		$i = 0;
		$lastval = '';
		$retval = array();
		
		// /a/b/c/d  a=>b,c=>d 第0 2 4…个作为key
		foreach ($segments as $seg)
		{
			if ($i % 2)
			{
				$retval[$lastval] = $seg;
			}
			else
			{
				$retval[$seg] = NULL;
				$lastval = $seg;
			}

			$i++;
		}

		if (count($default) > 0)
		{
			foreach ($default as $val)
			{
				if ( ! array_key_exists($val, $retval))
				{
					$retval[$val] = NULL;
				}
			}
		}

		// Cache the array for reuse
		isset($this->keyval[$which]) OR $this->keyval[$which] = array();
		$this->keyval[$which][$n] = $retval;
		return $retval;
	}

	// --------------------------------------------------------------------

	/**
	 * Assoc to URI
	 * 数组转uri
	 *
	 * Generates a URI string from an associative array.
	 *
	 * @param	array	$array	Input array of key/value pairs
	 * @return	string	URI string
	 */
	public function assoc_to_uri($array)
	{
		$temp = array();
		foreach ((array) $array as $key => $val)
		{
			$temp[] = $key;
			$temp[] = $val;
		}

		return implode('/', $temp);
	}

	// --------------------------------------------------------------------

	/**
	 * Slash segment
	 *
	 * Fetches an URI segment with a slash.
	 *
	 * @param	int	$n	Index
	 * @param	string	$where	Where to add the slash ('trailing' or 'leading')
	 * @return	string
	 */
	public function slash_segment($n, $where = 'trailing')
	{
		return $this->_slash_segment($n, $where, 'segment');
	}

	// --------------------------------------------------------------------

	/**
	 * Slash routed segment
	 *
	 * Fetches an URI routed segment with a slash.
	 *
	 * @param	int	$n	Index
	 * @param	string	$where	Where to add the slash ('trailing' or 'leading')
	 * @return	string
	 */
	public function slash_rsegment($n, $where = 'trailing')
	{
		return $this->_slash_segment($n, $where, 'rsegment');
	}

	// --------------------------------------------------------------------

	/**
	 * Internal Slash segment
	 *
	 * Fetches an URI Segment and adds a slash to it.
	 *
	 * @used-by	CI_URI::slash_segment()
	 * @used-by	CI_URI::slash_rsegment()
	 *
	 * @param	int	$n	Index
	 * @param	string	$where	Where to add the slash ('trailing' or 'leading')
	 * @param	string	$which	Array name ('segment' or 'rsegment')
	 * @return	string
	 */
	protected function _slash_segment($n, $where = 'trailing', $which = 'segment')
	{
		$leading = $trailing = '/';

		if ($where === 'trailing')
		{
			$leading	= '';
		}
		elseif ($where === 'leading')
		{
			$trailing	= '';
		}

		return $leading.$this->$which($n).$trailing;
	}

	// --------------------------------------------------------------------

	/**
	 * Segment Array
	 *
	 * @return	array	CI_URI::$segments
	 */
	public function segment_array()
	{
		return $this->segments;
	}

	// --------------------------------------------------------------------

	/**
	 * Routed Segment Array
	 *
	 * @return	array	CI_URI::$rsegments
	 */
	public function rsegment_array()
	{
		return $this->rsegments;
	}

	// --------------------------------------------------------------------

	/**
	 * Total number of segments
	 *
	 * @return	int
	 */
	public function total_segments()
	{
		return count($this->segments);
	}

	// --------------------------------------------------------------------

	/**
	 * Total number of routed segments
	 *
	 * @return	int
	 */
	public function total_rsegments()
	{
		return count($this->rsegments);
	}

	// --------------------------------------------------------------------

	/**
	 * Fetch URI string
	 *
	 * @return	string	CI_URI::$uri_string
	 */
	public function uri_string()
	{
		return $this->uri_string;
	}

	// --------------------------------------------------------------------

	/**
	 * Fetch Re-routed URI string
	 *
	 * @return	string
	 */
	public function ruri_string()
	{
		return ltrim(load_class('Router', 'core')->directory, '/').implode('/', $this->rsegments);
	}

}

```

