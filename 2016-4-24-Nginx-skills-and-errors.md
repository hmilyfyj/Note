title: Nginx 配置笔记
date: 2016-04-24 12:56
tags: [Nginx]
categories: Nginx
---


<!-- more -->

---

# 技巧

## SSL 配置

可以申请 `StratSSL` 家的免费证书。

需要新增的配置：

```php
 listen       443;
 ssl    on;
 ssl_certificate    /usr/local/nginx/conf/Startssl.crt;   #你从StartSSL下载证书放的路径
 ssl_certificate_key     /usr/local/nginx/conf/nicky1605.key;  #openssl生成key路径
 ssl_session_timeout 5m;
```

[SSL配置参考][1]
[参考网址2][2]

## 强制跳转 https

[参考地址][3]

## SSL 重启 nginx 免输密码

配置 SSL 后，重启 nginx 时将弹出 `Enter PEM pass phrase:` 的提示并等待。

解决方法如下：

第一步：

    openssl rsa -in server.key -out server.key.unsecure
    
第二步：

```
server {
  …………
  ssl_certificate /etc/nginx/certs/server.crt;
  ssl_certificate_key /etc/nginx/certs/server.key.unsecure;
  …………
}
```

    
[参考地址][4]

## 重启 Nginx 脚本

### 方法1：

使用 supervisor 时，直接：

    supervisorctl restart xxx(e.g:all)
    
### 方法2：

在 `/etc/init.d` 创建 `nginx` 文件，输入如下内容，并 `chmod +x nginx`

使用方法：`/etc/init.d/nginx {start|stop|reload|restart}`


```
#! /bin/sh
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
 
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DESC="nginx daemon"
NAME=nginx
DAEMON=/usr/local/nginx/sbin/$NAME
CONFIGFILE=/usr/local/nginx/conf/$NAME.conf
PIDFILE=/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
 

set -e
[ -x "$DAEMON" ] || exit 0
 
do_start() {
 $DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
 
do_stop() {
 kill -INT `cat $PIDFILE` || echo -n "nginx not running"
}
 
do_reload() {
 kill -HUP `cat $PIDFILE` || echo -n "nginx can't reload"
}
 
case "$1" in
 start)
 echo -n "Starting $DESC: $NAME"
 do_start
 echo "."
 ;;
 stop)
 echo -n "Stopping $DESC: $NAME"
 do_stop
 echo "."
 ;;
 reload|graceful)
 echo -n "Reloading $DESC configuration..."
 do_reload
 echo "."
 ;;
 restart)
 echo -n "Restarting $DESC: $NAME"
 do_stop
 do_start
 echo "."
 ;;
 *)
 echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
 exit 3
 ;;
esac
 
exit 0
```


## vhost 配置脚本

创建 `vhost.sh` 文件，并放入 `/usr/local/bin` ，并 `chmod +x vhost.sh`。
使用方法: `vhsot.sh create | delete domain`

```
#!/bin/bash
### Set Language
TEXTDOMAIN=virtualhost

### Set default parameters
action=$1
domain=$2
rootDir=$3
owner=$(who am i | awk '{print $1}')
sitesEnable='/etc/nginx/sites-enabled/'
sitesAvailable='/etc/nginx/sites-available/'
userDir='/data/www/'

if [ "$(whoami)" != 'root' ]; then
	echo $"You have no permission to run $0 as non-root user. Use sudo"
		exit 1;
fi

if [ "$action" != 'create' ] && [ "$action" != 'delete' ]
	then
		echo $"You need to prompt for action (create or delete) -- Lower-case only"
		exit 1;
fi

while [ "$domain" == "" ]
do
	echo -e $"Please provide domain. e.g.dev,staging"
	read domain
done

if [ "$rootDir" == "" ]; then
	rootDir=${domain//./}
fi

### if root dir starts with '/', don't use /var/www as default starting point
if [[ "$rootDir" =~ ^/ ]]; then
	userDir=''
fi

rootDir=$userDir$rootDir

if [ "$action" == 'create' ]
	then
		### check if domain already exists
		if [ -e $sitesAvailable$domain ]; then
			echo -e $"This domain already exists.\nPlease Try Another one"
			exit;
		fi

		### check if directory exists or not
		if ! [ -d $rootDir ]; then
			echo $rootDir
			exit
			### create the directory
			mkdir $rootDir
			### give permission to root dir
			chmod 755 $rootDir
			### write test file in the new domain dir
			if ! echo "<?php echo phpinfo(); ?>" > $rootDir/phpinfo.php
				then
					echo $"ERROR: Not able to write in file $userDir/$rootDir/phpinfo.php. Please check permissions."
					exit;
			else
					echo $"Added content to $rootDir/phpinfo.php."
			fi
		fi

		### create virtual host rules file
		if ! echo "server {
			listen   80;
			root $rootDir;
			index index.php index.html index.htm;
			server_name $domain;

			# serve static files directly
			location ~* \.(jpg|jpeg|gif|css|png|js|ico|html)$ {
				access_log off;
				expires max;
			}

			# removes trailing slashes (prevents SEO duplicate content issues)
			if (!-d \$request_filename) {
				rewrite ^/(.+)/\$ /\$1 permanent;
			}

			# unless the request is for a valid file (image, js, css, etc.), send to bootstrap
			if (!-e \$request_filename) {
				rewrite ^/(.*)\$ /index.php?/\$1 last;
				break;
			}

			# removes trailing 'index' from all controllers
			if (\$request_uri ~* index/?\$) {
				rewrite ^/(.*)/index/?\$ /\$1 permanent;
			}

			# catch all
			error_page 404 /index.php;

			location ~ \.php$ {
				fastcgi_split_path_info ^(.+\.php)(/.+)\$;
				fastcgi_pass 127.0.0.1:9000;
				fastcgi_index index.php;
				include fastcgi_params;
			}

			location ~ /\.ht {
				deny all;
			}

		}" > $sitesAvailable$domain
		then
			echo -e $"There is an ERROR create $domain file"
			exit;
		else
			echo -e $"\nNew Virtual Host Created\n"
		fi

		### Add domain in /etc/hosts
		# if ! echo "127.0.0.1	$domain" >> /etc/hosts
		# 	then
		# 		echo $"ERROR: Not able write in /etc/hosts"
		# 		exit;
		# else
		# 		echo -e $"Host added to /etc/hosts file \n"
		# fi

		if [ "$owner" == "" ]; then
			chown -R $(whoami):www-data $rootDir
		else
			chown -R $owner:www-data $rootDir
		fi

		### enable website
		ln -s $sitesAvailable$domain $sitesEnable$domain

		### restart Nginx
		service nginx restart

		### show the finished message
		echo -e $"Complete! \nYou now have a new Virtual Host \nYour new host is: http://$domain \nAnd its located at $rootDir"
		exit;
	else
		### check whether domain already exists
		if ! [ -e $sitesAvailable$domain ]; then
			echo -e $"This domain dont exists.\nPlease Try Another one"
			exit;
		else
			### Delete domain in /etc/hosts
			newhost=${domain//./\\.}
			sed -i "/$newhost/d" /etc/hosts

			### disable website
			rm $sitesEnable$domain

			### restart Nginx
			service nginx restart

			### Delete virtual host rules files
			rm $sitesAvailable$domain
		fi

		### check if directory exists or not
		if [ -d $rootDir ]; then
			echo -e $"Delete host root directory ? (s/n)"
			read deldir

			if [ "$deldir" == 's' -o "$deldir" == 'S' ]; then
				### Delete the directory
				rm -rf $rootDir
				echo -e $"Directory deleted"
			else
				echo -e $"Host directory conserved"
			fi
		else
			echo -e $"Host directory not found. Ignored"
		fi

		### show the finished message
		echo -e $"Complete!\nYou just removed Virtual Host $domain"
		exit 0;
fi

```


# 错误

## 1. 域名过长

### 报错

    could not build the server_names_hash, you should increase server_names_hash_bucket_size: 32

### 解决办法

在 nginx.conf 的 http 段添加：

    server_names_hash_bucket_size 64;
    
> 注：如果已经存在，需要加大后面的数值，注意：该数值是32的倍数为宜。

[参考地址][5]


  [1]: http://www.cnblogs.com/dasn/articles/4042506.html
  [2]: https://s.how/nginx-ssl/
  [3]: http://www.07net01.com/program/61139.html
  [4]: http://blog.526net.com/?p=2702
  [5]: http://www.cnblogs.com/yun007/p/3739182.html