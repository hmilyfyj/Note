title: 手动配置lnmp环境
date: 2016-03-14 16:15
tags: [折腾,Linux,Mysql,Nginx]
categories: 
---

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

	wget http://nginx.org/download/nginx-1.6.2.tar.gz tar -xvf nginx-1.6.2.tar.gz





