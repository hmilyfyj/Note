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
- 引入配置文件，根据数据库配置来引入适配器：`CI_DB` 、`CI_DB_driver`、`CI_DB_query_builder`。然后实例化 CI_DB_mysqli_driver。
- `CI_DB` 类是临时声明的，它根据`$query_builder` 参数决定 `class CI_DB extends CI_DB_driver { } `  还是 `class CI_DB extends CI_DB_query_builder { }`,
- `CI_DB_query_builder ` 继承自`CI_DB`

继续跟进 `$DB->initialize();`

```php
/**
	 * Initialize Database Settings
	 *
	 * @return	bool
	 */
	public function initialize()
	{
		/* If an established connection is available, then there's
		 * no need to connect and select the database.
		 *
		 * Depending on the database driver, conn_id can be either
		 * boolean TRUE, a resource or an object.
		 */
		if ($this->conn_id)
		{
			return TRUE;
		}

		// ----------------------------------------------------------------

		// 尝试连接数据库
		// Connect to the database and set the connection ID
		$this->conn_id = $this->db_connect($this->pconnect);

		// No connection resource? Check if there is a failover else throw an error
		if ( ! $this->conn_id)
		{
			// Check if there is a failover set 连接失败时的备用配置
			if ( ! empty($this->failover) && is_array($this->failover))
			{
				// Go over all the failovers
				foreach ($this->failover as $failover)
				{
					// Replace the current settings with those of the failover
					foreach ($failover as $key => $val)
					{
						$this->$key = $val;
					}

					// Try to connect
					$this->conn_id = $this->db_connect($this->pconnect);

					// If a connection is made break the foreach loop
					if ($this->conn_id)
					{
						break;
					}
				}
			}

			// We still don't have a connection?
			// 执行完 PLAN abcdefg 仍然没连上，报错
			if ( ! $this->conn_id)
			{
				log_message('error', 'Unable to connect to the database');

				if ($this->db_debug)
				{
					$this->display_error('db_unable_to_connect');
				}

				return FALSE;
			}
		}

		// Now we set the character set and that's all
		// 设定编码
		return $this->db_set_charset($this->char_set);
	}
```

这一步进行了数据库的连接和编码的设置。

继续跟进 `$this->db_connect($this->pconnect);`

```php
/**
	 * Database connection
	 *
	 * @param	bool	$persistent
	 * @return	object
	 */
	public function db_connect($persistent = FALSE)
	{
		// Do we have a socket path?
		if ($this->hostname[0] === '/')
		{
			$hostname = NULL;
			$port = NULL;
			$socket = $this->hostname;
		}
		else
		{
			// Persistent(持久化) connection support was added in PHP 5.3.0
			$hostname = ($persistent === TRUE && is_php('5.3'))
				? 'p:'.$this->hostname : $this->hostname;
			$port = empty($this->port) ? NULL : $this->port;
			$socket = NULL;
		}

		//使用压缩协议
		$client_flags = ($this->compress === TRUE) ? MYSQLI_CLIENT_COMPRESS : 0;
		$mysqli = mysqli_init();

		// 查询超时
		$mysqli->options(MYSQLI_OPT_CONNECT_TIMEOUT, 10);

		if ($this->stricton)
		{
			//参考地址 http://www.jb51.net/article/51900.htm
			$mysqli->options(MYSQLI_INIT_COMMAND, 'SET SESSION sql_mode="STRICT_ALL_TABLES"');
		}

		if (is_array($this->encrypt))
		{
			$ssl = array();
			empty($this->encrypt['ssl_key'])    OR $ssl['key']    = $this->encrypt['ssl_key'];
			empty($this->encrypt['ssl_cert'])   OR $ssl['cert']   = $this->encrypt['ssl_cert'];
			empty($this->encrypt['ssl_ca'])     OR $ssl['ca']     = $this->encrypt['ssl_ca'];
			empty($this->encrypt['ssl_capath']) OR $ssl['capath'] = $this->encrypt['ssl_capath'];
			empty($this->encrypt['ssl_cipher']) OR $ssl['cipher'] = $this->encrypt['ssl_cipher'];

			if ( ! empty($ssl))
			{
				if ( ! empty($this->encrypt['ssl_verify']) && defined('MYSQLI_OPT_SSL_VERIFY_SERVER_CERT'))
				{
					$mysqli->options(MYSQLI_OPT_SSL_VERIFY_SERVER_CERT, TRUE);
				}

				$client_flags |= MYSQLI_CLIENT_SSL;
				$mysqli->ssl_set(
					isset($ssl['key'])    ? $ssl['key']    : NULL,
					isset($ssl['cert'])   ? $ssl['cert']   : NULL,
					isset($ssl['ca'])     ? $ssl['ca']     : NULL,
					isset($ssl['capath']) ? $ssl['capath'] : NULL,
					isset($ssl['cipher']) ? $ssl['cipher'] : NULL
				);
			}
		}

		if ($mysqli->real_connect($hostname, $this->username, $this->password, $this->database, $port, $socket, $client_flags))
		{
			// 开启了 ssl 并且 mysql 版本小于 5.7.3 时，加密传输将失效。
			// Prior to version 5.7.3, MySQL silently downgrades to an unencrypted connection if SSL setup fails
			if (
				($client_flags & MYSQLI_CLIENT_SSL)
				&& version_compare($mysqli->client_info, '5.7.3', '<=')
				&& empty($mysqli->query("SHOW STATUS LIKE 'ssl_cipher'")->fetch_object()->Value)
			)
			{
				$mysqli->close();
				$message = 'MySQLi was configured for an SSL connection, but got an unencrypted connection instead!';
				log_message('error', $message);
				return ($this->db->db_debug) ? $this->db->display_error($message, '', TRUE) : FALSE;
			}

			return $mysqli;
		}

		return FALSE;
	}
```

根据配置的参数尽兴连接，并返回连接实例或这报错。

