title: 人生苦短，学学 Docker
date: 2016-04-22 18:38
tags: [折腾,Docker]
categories: Docker
---

饱尝部署环境之麻烦，决定花点时间学 Docker，看看能不能偷懒 & 一劳永逸。

<!-- more -->

---

# 总结下需要的环境：

1.mysql
2.php7
3.nginx
4.composer/laravel

# 常用命令

交互命令：

    docker run -i -t  daocloud.io/hmilyfyj/memcached:master-a9a55bf  /bin/bash
    
大扫除（清除所有容器 && 镜像）：

      docker kill \$(docker ps -q); docker rm \$(docker ps -a -q)
      docker rmi \$(docker images -q -a) 
      
交互

1.获取PID

    PID=\$(docker inspect --format "\{\{ \.State\.Pid \}\}" <container-id>)

2.

    nsenter --target \$PID --mount --uts --ipc --net --pid
    
组合起来：

    PID=\$(docker inspect --format "\{\{ .State.Pid \}\}" <container-id>) && nsenter --target \$PID --mount --uts --ipc --net --pid
    
3.

    docker run -d -p 80:80 --name web --link dao_memcached_1:memcached  daocloud.io/hmilyfyj/php7_nginx:master-4e74f55
    
    
    

# 配置 Laravel 环境

1.安装 Composer

sudo composer config -l -g

```shell
curl -sS https://getcomposer.org/installer \
        | php -- --install-dir=/usr/local/bin --filename=composer
```

2.Laravel

    composer global require "laravel/installer"

3.Dumper


# memcached 扩展

在 tmp 目录下载
1.wget http://pecl.php.net/get/memcache-3.0.8.tgz
2.COPY libevent-2.0.22-stable.tar.gz /tmp/libevent-2.0.22-stable.tar.gz

```Shell
cd /tmp
tar zxvf libevent-2.0.22-stable.tar
cd libevent-2.0.22-stable
./configure --prefix=/usr/local
make; make install

cd /tmp && tar zxvf libevent-2.0.22-stable.tar && cd libevent-2.0.22-stable && ./configure --prefix=/usr/local && make && make install
```


http://www.open-open.com/lib/view/open1451625537245.html
lib
```shell
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz

cd /tmp && wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz && tar -zxvf libmemcached-1.0.18.tar.gz && cd libmemcached-1.0.18 && ./configure --prefix=/usr/local/libmemcached && make && make install
```

mem
```php
git clone https://github.com/php-memcached-dev/php-memcached -b php7 && cd php-memcached && phpize && ./configure --with-libmemcached-dir=/usr/local/libmemcached --with-php-config=/usr/local/php/bin/php-config  --disable-memcached-sasl && make && make install

./configure --with-libmemcached-dir=/usr/local/libmemcached --with-php-config=/usr/local/php/bin/php-config  --disable-memcached-sasl

```


```Shell
cd /tmp  #首先进入到该下载包的目录
tar zxvf memcache-3.0.8.tgz #解压包
cd memcache-3.0.8 #进入到解压的目录
/opt/lampp/bin/phpize #动态为php添加扩展。phpize路径可能不一致，请根据自己的实际情况
./configure –enable-memcache –with-php-config=/opt/lampp/bin/php-config –with-zlib-dir #php-config请根据自己环境情况填写
make; make install #编译+安装

cd /tmp && tar zxvf memcache-3.0.8.tgz && cd memcache-3.0.8 && /usr/local/php/bin/phpize && ./configure -enable-memcache -with-php-config=/usr/local/php/bin/php-config -with-zlib-dir && make && make install


./configure -enable-memcache -with-php-config=/usr/local/php/bin/php-config -with-zlib-dir
```

http://pkg.phpcomposer.com/
