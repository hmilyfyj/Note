title: Centos下 Memcached 的安装
date: 2015-11-28 21:48:02
categories: Centos
tags: [折腾,Linux,Centos,Memcached]
---

## 下载

```
cd /tmp
# wget http://www.danga.com/memcached/dist/memcached-1.2.0.tar.gz
# wget http://www.monkey.org/~provos/libevent-1.2.tar.gz
```

## 安装 gcc、libevent

```
//gcc
yum -y install gcc
yum -y install gcc-c++

//libevent
tar xzvf libevent-2.0.21-stable.tar.gz ##解压
cd libevent-2.0.21-stable
./configure --prefix=/usr
make
make install

//memcached
 cd /tmp
tar zxvf memcached-1.2.0.tar.gz
cd memcached-1.2.0
./configure –with-libevent=/usr
make
make install
```

<!--more-->

## memcahced：

> -d选项是启动一个守护进程，
-m是分配给Memcache使用的内存数量，单位是MB，我这里是10MB，
-u是运行Memcache的用户，我这里是root，
-l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.0.200，
-p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口，
-c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定，
-P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid，



## 启动 memcached

    /usr/local/bin/memcached -d -m 1024 -u root -p 11211 -P /tmp/memcached.pid
