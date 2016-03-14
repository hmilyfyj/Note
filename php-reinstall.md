title: 手动配置lnmp环境
date: 2016-03-14 16:15
tags: [折腾,Linux,Mysql,Nginx]
categories: 折腾
---

使用一键安装脚本的确方便，但无法自定义参数，比如我想加入gg反代，但需要的扩展功能需要手动添加，然而..我只会用一键安装...

所以有了这一篇记录。

<!-- more -->

---

# Nginx


## 删除老版本

    [root@bfmbch /]# find / -name nginx
	/root/hexo_blog/.deploy_git/categories/nginx
	/root/hexo_blog/.deploy_git/tags/nginx
	/root/hexo_blog/public/categories/nginx
	/root/hexo_blog/public/tags/nginx
	/etc/rc.d/init.d/nginx
	/usr/bin/nginx
	/usr/local/nginx
	/usr/local/nginx/sbin/nginx

	rm -rf /usr/bin/nginx
	rm -rf /usr/local/ngin
	rm -rf /usr/local/nginx/sbin/nginx

# 安装 gcc & git

    yum install build-essential git gcc gcc-c++ make

# 下载，编译与安装

	# 下载最新版源码
	# nginx 官网: 
	# http://nginx.org/en/download.html
	#
	wget "http://nginx.org/download/nginx-1.9.12.tar.gz"

	#
	# 下载最新版 pcre
	# pcre 官网:
	# http://www.pcre.org/
	#
	wget "ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz"

	#
	# 下载最新版 openssl
	# opessl 官网:
	# https://www.openssl.org/
	#
	wget "https://www.openssl.org/source/openssl-1.0.1j.tar.gz"

	#
	# 下载最新版 zlib
	# zlib 官网:
	# http://www.zlib.net/
	#
	wget "http://zlib.net/zlib-1.2.8.tar.gz"

	#
	# 下载google扩展
	#
	git clone https://github.com/cuber/ngx_http_google_filter_module

	#
	# 下载 substitutions 扩展
	#
	git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module

	# 解压缩

	tar xzvf nginx-1.9.12.tar.gz
	tar xzvf pcre-8.38.tar.gz
	tar xzvf openssl-1.0.1j.tar.gz
	tar xzvf zlib-1.2.8.tar.gz

	# 编译与安装

	# 进入 nginx 源码目录
	#
	cd nginx-1.9.12

	#
	# 设置编译选项
	#
	./configure \
	  --prefix=/opt/nginx-1.9.12 \
	  --with-pcre=../pcre-8.38 \
	  --with-openssl=../openssl-1.0.1j \
	  --with-zlib=../zlib-1.2.8 \
	  --with-http_ssl_module \
	  --add-module=../ngx_http_google_filter_module \
	  --add-module=../ngx_http_substitutions_filter_module

	make
	make install

	# 使用root账号启动

	 /opt/nginx-1.9.12/sbin/nginx



	vi /opt/nginx-1.7.8/conf/nginx.conf

在配置文件末尾加上

	include /etc/nginx/sites-enabled/*;

    mkdir -p /etc/nginx/sites-enabled
    mkdir -p /etc/nginx/sites-available
    mkdir -p /var/log/nginx
    mkdir -p /var/cache/nginx/cache
    mkdir -p /var/cache/nginx/temp
	
其中http的配置方式

	server {
	  server_name g.qing.ga;
	  listen 80;

	  resolver 8.8.8.8;
	  location / {
	    google on;
	  }
	}

# PHP

## 删除老版本

	[root@bfmbch ~]# find / -name php
	/root/hexo_blog/.deploy_git/tags/php
	/root/hexo_blog/public/tags/php
	/usr/bin/php
	/usr/local/php
	/usr/local/php/lib/php
	/usr/local/php/bin/php
	/usr/local/php/include/php
	/usr/local/php/php
	/usr/local/php/php/php

	rm -rf /usr/bin/php
	rm -rf /usr/local/php

## 安装

	./buildconf --force
	./configure --prefix=/apps/php/php7.0 --enable-mbstring --with-curl --with-gd --with-config-file-path=/apps/php/php7.0/etc/ --enable-fpm --enable-mysqlnd --with-pdo-mysql=mysqlnd --disable-fileinfo
	make && make install
### 报错

错误1：

	ext/iconv/.libs/iconv.o: In function `php_iconv_stream_filter_ctor':
	/home/king/php-5.2.13/ext/iconv/iconv.c:2491: undefined reference to `libiconv_open'
	collect2: ld returned 1 exit status
	make: *** [sapi/cli/php] Error 1
	[root@test php-5.2.13]# vi Makefile

解决办法：

	make ZEND_EXTRA_LIBS='-liconv'

错误2：
	
	cc: Internal error: Killed (program cc1)

解决办法：

>先停用内存占用大的程序，重新configure，make，如果还是报错，在configure的时候添加--disable-fileinfo参数

[参考地址](http://blog.sina.com.cn/s/blog_48ab118d0102v5fn.html)




