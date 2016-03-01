title: CI-数据库操作源码分析
date: 2016-03-01 13:17
tags: [CodeIgniter,PHP]
categories: CodeIgniter
---

Model层牵涉到数据的增删改查，很重要，也是较为薄弱的地方，在这里研读CI数据库相关的代码。

<!-- more -->

---

# 一切数据操作的源头

    $this->load->database();

这句调用了`Loader`类的 `database()` 函数，跟进：

```php
/**
	 * Database Loader
	 *
	 * @param	mixed	$params		Database configuration options
	 * @param	bool	$return 	Whether to return the database object
	 * @param	bool	$query_builder	Whether to enable Query Builder
	 *					(overrides the configuration setting)
	 *
	 * @return	object|bool	Database object if $return is set to TRUE,
	 *					FALSE on failure, CI_Loader instance in any other case
	 */
	public function database($params = '', $return = FALSE, $query_builder = NULL)
	{
		// Grab the super object
		// 获取实例
		$CI =& get_instance();

		// Do we even need to load the database class?
		// 已经载入过且return 为 false,不再载入。
		if ($return === FALSE && $query_builder === NULL && isset($CI->db) && is_object($CI->db) && ! empty($CI->db->conn_id))
		{
			return FALSE;
		}

		//引入文件，内含 DB 函数
		require_once(BASEPATH.'database/DB.php');

		//返回数据库连接实例
		if ($return === TRUE)
		{
			return DB($params, $query_builder);
		}

		// Initialize the db variable. Needed to prevent
		// reference errors with some configurations
		$CI->db = '';

		//调用 DB() 函数创建数据库对象并创建引用。
		// Load the DB class
		$CI->db =& DB($params, $query_builder);
		return $this;
	}
``` 

 接着调用了`DB.php` 文件的 `DB()`函数，跟进
 
```php
/**
 * Initialize the database
 *
 * @category	Database
 * @author	EllisLab Dev Team
 * @link	http://codeigniter.com/user_guide/database/
 *
 * @param 	string|string[]	$params
 * @param 	bool		$query_builder_override
 *				Determines if query builder should be used or not
 */
function &DB($params = '', $query_builder_override = NULL)
{
	// Load the DB config file if a DSN string wasn't passed
	// dsn为不包含 ://字符串。
	if (is_string($params) && strpos($params, '://') === FALSE)
	{
		// Is the config file in the environment folder?
		if ( ! file_exists($file_path = APPPATH.'config/'.ENVIRONMENT.'/database.php')
			&& ! file_exists($file_path = APPPATH.'config/database.php'))
		{
			show_error('The configuration file database.php does not exist.');
		}

		// 引入配置文件
		include($file_path);

		// Make packages contain database config files,
		// given that the controller instance already exists
		if (class_exists('CI_Controller', FALSE))
		{
			foreach (get_instance()->load->get_package_paths() as $path)
			{
				if ($path !== APPPATH)
				{
					if (file_exists($file_path = $path.'config/'.ENVIRONMENT.'/database.php'))
					{
						include($file_path);
					}
					elseif (file_exists($file_path = $path.'config/database.php'))
					{
						include($file_path);
					}
				}
			}
		}
		
		//检测配置文件
		if ( ! isset($db) OR count($db) === 0)
		{
			show_error('No database connection settings were found in the database config file.');
		}

		//$parmas指定了数据库配置组名
		if ($params !== '')
		{
			$active_group = $params;
		}

		if ( ! isset($active_group))
		{
			show_error('You have not specified a database connection group via $active_group in your config/database.php file.');
		}
		elseif ( ! isset($db[$active_group]))
		{
			show_error('You have specified an invalid database connection group ('.$active_group.') in your config/database.php file.');
		}

		// 将最终指定的数据库配置赋值给 $params 参数
		$params = $db[$active_group];
	}
	elseif (is_string($params))
	{
		/**
		 * Parse the URL from the DSN string
		 * Database settings can be passed as discreet
		 * parameters or as a data source name in the first
		 * parameter. DSNs must have this prototype:
		 * $dsn = 'driver://username:password@hostname/database';
		 */
		if (($dsn = @parse_url($params)) === FALSE)
		{
			show_error('Invalid DB Connection String');
		}

		$params = array(
			'dbdriver'	=> $dsn['scheme'],
			'hostname'	=> isset($dsn['host']) ? rawurldecode($dsn['host']) : '',
			'port'		=> isset($dsn['port']) ? rawurldecode($dsn['port']) : '',
			'username'	=> isset($dsn['user']) ? rawurldecode($dsn['user']) : '',
			'password'	=> isset($dsn['pass']) ? rawurldecode($dsn['pass']) : '',
			'database'	=> isset($dsn['path']) ? rawurldecode(substr($dsn['path'], 1)) : ''
		);

		// Were additional config items set?
		if (isset($dsn['query']))
		{
			parse_str($dsn['query'], $extra);

			foreach ($extra as $key => $val)
			{
				if (is_string($val) && in_array(strtoupper($val), array('TRUE', 'FALSE', 'NULL')))
				{
					$val = var_export($val, TRUE);
				}

				$params[$key] = $val;
			}
		}
	}

	// No DB specified yet? Beat them senseless...
	if (empty($params['dbdriver']))
	{
		show_error('You have not selected a database type to connect to.');
	}

	// CI 为了兼容不同的数据库，所以要通过用户指定数据库类型
	// Load the DB classes. Note: Since the query builder class is optional
	// we need to dynamically create a class that extends proper parent class
	// based on whether we're using the query builder class or not.
	if ($query_builder_override !== NULL)
	{
		$query_builder = $query_builder_override;
	}
	// Backwards compatibility work-around（应急兼容性） for keeping the
	// $active_record config variable working. Should be
	// removed in v3.1
	// 通过设定 $query_builder 参数来指定数据库配置
	elseif ( ! isset($query_builder) && isset($active_record))
	{
		
		$query_builder = $active_record;
	}
	
	//引入 CI_DB_driver 类。
	require_once(BASEPATH.'database/DB_driver.php');

	if ( ! isset($query_builder) OR $query_builder === TRUE)
	{
		//查询构造器
		require_once(BASEPATH.'database/DB_query_builder.php');
		
		// 创建 CI_DB 类
		if ( ! class_exists('CI_DB', FALSE))
		{
			/**
			 * CI_DB
			 *
			 * Acts as an alias for both CI_DB_driver and CI_DB_query_builder.
			 *
			 * @see	CI_DB_query_builder
			 * @see	CI_DB_driver
			 */
			class CI_DB extends CI_DB_query_builder { }
		}
	}
	elseif ( ! class_exists('CI_DB', FALSE))
	{
		/**
		 * 不再引入 CI_DB_query_builder类
		 * 
	 	 * @ignore
		 */
		class CI_DB extends CI_DB_driver { }
	}

	// Load the DB driver
	$driver_file = BASEPATH.'database/drivers/'.$params['dbdriver'].'/'.$params['dbdriver'].'_driver.php';

	// 引入指定数据库类型的驱动函数
	file_exists($driver_file) OR show_error('Invalid DB driver');
	require_once($driver_file);

	// Instantiate the DB adapter
	$driver = 'CI_DB_'.$params['dbdriver'].'_driver';
	$DB = new $driver($params);

	// Check for a subdriver
	// PDO
	if ( ! empty($DB->subdriver))
	{
		$driver_file = BASEPATH.'database/drivers/'.$DB->dbdriver.'/subdrivers/'.$DB->dbdriver.'_'.$DB->subdriver.'_driver.php';

		if (file_exists($driver_file))
		{
			require_once($driver_file);
			$driver = 'CI_DB_'.$DB->dbdriver.'_'.$DB->subdriver.'_driver';
			$DB = new $driver($params);
		}
	}

	//初始化
	$DB->initialize();
	return $DB;
}
```
引入配置文件，根据数据库配置来引入适配器：`CI_DB` 、`CI_DB_driver`、`CI_DB_query_builder`。然后实例化 CI_DB_mysqli_driver。
类是临时生成的，它根据`$query_builder` 参数决定 `class CI_DB extends CI_DB_driver { } `  还是 `class CI_DB extends CI_DB_query_builder { }`


