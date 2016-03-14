title: Nginx、Codeigniter 的结合
date: 2015-11-29 11:24:26
tags: [折腾,CodeIgniter,Nginx]
categories: nginx
---

### .conf 的配置

    server {
    listen       80;
    server_name  my.xuyu.club;

    #charset utf-8;
    #access_log  /var/log/nginx/log/host.access.log  main;
	root   /usr/share/nginx/html;
	index  index.php index.html index.htm;
	
	location / {
    try_files $uri $uri/ /index.php?/$request_uri;
	}
	
	location ~* ^/(assets|files|robots\.txt) { }
	
	location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name;
        include        fastcgi_params;
    }
	
	location ~ /\.ht {
        deny  all;
    }
    }



