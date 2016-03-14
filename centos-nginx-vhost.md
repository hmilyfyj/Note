title: Centos、nginx 手动配置 vhost
date: 2015-12-3 21:51:55
tags: [折腾,Centos,Nginx]
categories: Nginx
---

今天很流畅的打出了 rm -rf /*，终于成了我嘲笑过的人。。

#### 创建文件夹

    cd /etc/nginx
    mkdir sites-available
    mkdir sites-enabled

#### 创建配置文件
    
    vi sites-available/xxxx.conf
    
<!--more-->

##### 写入server配置（这个是配合 CI 的配置）：

    server {
    listen       80;
    server_name  www.xuyu.club www.fengbl.cn;
    
    #charset utf-8;
    #access_log  /var/log/nginx/log/host.access.log  main;
    root   /var/www/www.fengbl.cn;
    index  index.php index.html index.htm;
    
    location / {
    try_files $uri $uri/ /index.php?/$request_uri;
    }
    
    location ~* ^/(assets|files|robots\.txt) { }
    
    location ~ \.php$ {
        root           /var/www/www.fengbl.cn;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name;
        include        fastcgi_params;
    }
    
    location ~ /\.ht {
        deny  all;
    }
    }

#### 创建软连接

    ln -s /etc/nginx/sites-available/xxxx.conf /etc/nginx/sites-enabled/xxxx.conf
    
#### 修改 nginx.conf 加入：

    include /etc/nginx/sites-enabled/*
    
#### 重启 nginx
    
    service nginx restart
    
大功告成
    