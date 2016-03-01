title: CI-数据库操作源码分析
date: 2016-03-01 13:17
tags: [CodeIgniter,PHP]
categories: CodeIgniter
---

Model层牵涉到数据的增删改查，很重要，也是我较为薄弱的地方，在这里研读CI数据库相关的代码。

<!-- more -->

---

# 未解决问题

SSL、compress 相关问题

# Tips
## 数据库连时

## 查询

无条件的delete查询在mysql4.1.2 版之前mysql_affected_rows为0。

# 流程总结

## 初始化
1. 引入基本类 `CI_DB_driver`基类、`CI_DB_query_builder`基类
2. 创建类`CI_DB`，且`CI_DB`根据 $query_builder 的T/F值继承 `CI_DB_driver`基类、`CI_DB_query_builder`基类。
3. 根据数据库类型（这里指定了 mysli）引入以`CI_DB`为基类的`CI_DB_mysqli_driver`  适配器。
4.  实例化 `CI_DB_mysqli_driver`

### 实例化
1.  调用 `db_connect()` 函数，实际上执行了：`mysqli_init();`
2. 做一些连接前准备，如`$mysqli->options(MYSQLI_OPT_CONNECT_TIMEOUT, 10);`、Strit mode `$mysqli->options(MYSQLI_INIT_COMMAND, 'SET SESSION sql_mode="STRICT_ALL_TABLES"');`
3. 创建连接`$mysqli->real_connect($hostname, $this->username, $this->password, $this->database, $port, $socket, $client_flags)`
4. 设置编码 `set_charset($charset)`

## 执行


# 详细代码

## 初始化

    $this->load->database();

>这句调用了`Loader`类的 `database()` 函数，

跟进：


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

 >接着调用了DB.php 文件的 `DB()`函数，跟进`DB()`函数：


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

>这一步进行了数据库的连接和编码的设置。

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
				//按位或
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

>根据配置的参数进行连接，并返回连接实例或这报错。


## 执行 sql 语句

    $sql     = "SELECT * FROM web_admins WHERE username = ? ";
    $query   = $this->db->query($sql, array($username));

跟进 query()：


```php
/**
	 * Execute the query
	 *
	 * Accepts an SQL string as input and returns a result object upon
	 * successful execution of a "read" type query. Returns boolean TRUE
	 * upon successful execution of a "write" type query. Returns boolean
	 * FALSE upon failure, and if the $db_debug variable is set to TRUE
	 * will raise an error.
	 *
	 * @param	string	$sql
	 * @param	array	$binds = FALSE		An array of binding data
	 * @param	bool	$return_object = NULL
	 * @return	mixed
	 */
	public function query($sql, $binds = FALSE, $return_object = NULL)
	{
		if ($sql === '')
		{
			log_message('error', 'Invalid query: '.$sql);
			return ($this->db_debug) ? $this->display_error('db_invalid_query') : FALSE;
		}
		elseif ( ! is_bool($return_object))
		{
			$return_object = ! $this->is_write_type($sql);
		}

		// Verify table prefix and replace if necessary
		if ($this->dbprefix !== '' && $this->swap_pre !== '' && $this->dbprefix !== $this->swap_pre)
		{
			$sql = preg_replace('/(\W)'.$this->swap_pre.'(\S+?)/', '\\1'.$this->dbprefix.'\\2', $sql);
		}

		// Compile binds if needed
		if ($binds !== FALSE)
		{
			//对要绑定的变量尽兴转义、检测
			$sql = $this->compile_binds($sql, $binds);
		}

		// Is query caching enabled? If the query is a "read type"
		// we will load the caching class and return the previously
		// cached query if it exists
		if ($this->cache_on === TRUE && $return_object === TRUE && $this->_cache_init())
		{
			$this->load_rdriver();
			if (FALSE !== ($cache = $this->CACHE->read($sql)))
			{
				return $cache;
			}
		}

		// Save the query for debugging
		if ($this->save_queries === TRUE)
		{
			//记录执行语句
			$this->queries[] = $sql;
		}

		// Start the Query Timer
		$time_start = microtime(TRUE);

		// Run the Query
		if (FALSE === ($this->result_id = $this->simple_query($sql)))
		{
			if ($this->save_queries === TRUE)
			{
				$this->query_times[] = 0;
			}

			// This will trigger（引发） a rollback if transactions are being used
			if ($this->_trans_depth !== 0)
			{
				$this->_trans_status = FALSE;
			}

			// Grab the error now, as we might run some additional queries before displaying the error
			$error = $this->error();

			// Log errors
			log_message('error', 'Query error: '.$error['message'].' - Invalid query: '.$sql);

			if ($this->db_debug)
			{
				// We call this function in order to roll-back queries
				// if transactions are enabled. If we don't call this here
				// the error message will trigger an exit, causing the
				// transactions to remain in limbo.
				if ($this->_trans_depth !== 0)
				{
					do
					{
						$this->trans_complete();
					}
					while ($this->_trans_depth !== 0);
				}

				// Display errors
				return $this->display_error(array('Error Number: '.$error['code'], $error['message'], $sql));
			}

			return FALSE;
		}

		// Stop and aggregate the query time results
		$time_end = microtime(TRUE);
		$this->benchmark += $time_end - $time_start;

		if ($this->save_queries === TRUE)
		{
			$this->query_times[] = $time_end - $time_start;
		}

		// Increment the query counter
		$this->query_count++;

		// Will we have a result object instantiated? If not - we'll simply return TRUE
		if ($return_object !== TRUE)
		{
			// If caching is enabled we'll auto-cleanup any existing files related to this particular URI
			if ($this->cache_on === TRUE && $this->cache_autodel === TRUE && $this->_cache_init())
			{
				$this->CACHE->delete();
			}

			return TRUE;
		}

		// Load and instantiate the result driver
		// 例如：CI_DB_Mysli_result
		$driver		= $this->load_rdriver();
		$RES		= new $driver($this);

		// Is query caching enabled? If so, we'll serialize the
		// result object and save it to a cache file.
		if ($this->cache_on === TRUE && $this->_cache_init())
		{
			// We'll create a new instance of the result object
			// only without the platform specific driver since
			// we can't use it with cached data (the query result
			// resource ID won't be any good once we've cached the
			// result object, so we'll have to compile the data
			// and save it)
			$CR = new CI_DB_result($this);
			$CR->result_object	= $RES->result_object();
			$CR->result_array	= $RES->result_array();
			$CR->num_rows		= $RES->num_rows();

			// Reset these since cached objects can not utilize resource IDs.
			$CR->conn_id		= NULL;
			$CR->result_id		= NULL;

			$this->CACHE->write($sql, $CR);
		}

		return $RES;
	}
``` 
>以上代码对 sql 语句 和 要绑定的内容进行了检测、和绑定，并通过调用 的方式调用不同数据库类型所属驱动的`simple_query()`方法进行查询，这里分别跟进`compile_binds()`、`simple_query()` 函数 。


 **compile_binds() 函数实现**：

```php
/**
	 * Compile Bindings
	 *
	 * @param	string	the sql statement
	 * @param	array	an array of bind data
	 * @return	string
	 */
	public function compile_binds($sql, $binds)
	{
		//处理不同类型、格式的参数
		if (empty($binds) OR empty($this->bind_marker) OR strpos($sql, $this->bind_marker) === FALSE)
		{
			// 没有要绑定的标记、参数为空则直接返回参数
			return $sql;
		}
		elseif ( ! is_array($binds))
		{
			$binds = array($binds);
			$bind_count = 1;
		}
		else
		{
			// Make sure we're using numeric keys
			// 保证参数key为数字、计算参数个数
			$binds = array_values($binds);
			$bind_count = count($binds);
		}

		// We'll need the marker length later
		// 我们使用 ？ ，所以这里 $ml=1;
		$ml = strlen($this->bind_marker);
		
		// 防止数据被我们错误替换
		// Make sure not to replace a chunk(数据块) inside a string that happens to match the bind marker
		// 为了保证替换时不替换数据块，这里先处理掉 '' 包裹的 ? 后再统计了 ? 的位置
		if ($c = preg_match_all("/'[^']*'/i", $sql, $matches))
		{
			$c = preg_match_all('/'.preg_quote($this->bind_marker, '/').'/i',
				str_replace($matches[0],
					str_replace($this->bind_marker, str_repeat(' ', $ml), $matches[0]), // bind_marker转 ' '
					$sql, $c),
				$matches, PREG_OFFSET_CAPTURE);
		
			// Bind values' count must match the count of markers in the query
			// 数量不匹配，返回原参数。
			if ($bind_count !== $c)
			{
				return $sql;
			}
		}
		elseif (($c = preg_match_all('/'.preg_quote($this->bind_marker, '/').'/i', $sql, $matches, PREG_OFFSET_CAPTURE)) !== $bind_count)
		{
			return $sql;
		}

		do
		{
			$c--;
			$escaped_value = $this->escape($binds[$c]);
			if (is_array($escaped_value))
			{
				$escaped_value = '('.implode(',', $escaped_value).')';
			}
			$sql = substr_replace($sql, $escaped_value, $matches[0][$c][1], $ml);
		}
		while ($c !== 0);

		return $sql;
	}
```

 **simple_query()函数实现**：
```php
/**
	 * Simple Query
	 * This is a simplified version of the query() function. Internally
	 * we only use it when running transaction commands since they do
	 * not require all the features of the main query() function.
	 *
	 * @param	string	the sql query
	 * @return	mixed
	 */
	public function simple_query($sql)
	{
		if ( ! $this->conn_id)
		{
			$this->initialize();
		}
		//_execute该函数由继承者各自实现
		return $this->_execute($sql);
	}
```

跟进 `_execute()` 函数：

```php
/**
	 * Execute the query
	 *
	 * @param	string	$sql	an SQL query
	 * @return	mixed
	 */
	protected function _execute($sql)
	{
		return $this->conn_id->query($this->_prep_query($sql));
	}


/**
	 * Prep the query
	 *
	 * If needed, each database adapter can prep the query string
	 *
	 * @param	string	$sql	an SQL query
	 * @return	string
	 */
	protected function _prep_query($sql)
	{
		// 针对这样的语句 DELETE FROM TABLE affected rows 为 0；
		// mysqli_affected_rows() returns 0 for "DELETE FROM TABLE" queries. This hack
		// modifies the query so that it a proper number of affected rows is returned.
		if ($this->delete_hack === TRUE && preg_match('/^\s*DELETE\s+FROM\s+(\S+)\s*$/i', $sql))
		{
			return trim($sql).' WHERE 1=1';
		}

		return $sql;
	}

```



