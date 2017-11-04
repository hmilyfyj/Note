title: 人生苦短，学学 Docker (杂乱，暂未整理)
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

##容器交互

### 方法1：

    docker run -i -t  daocloud.io/hmilyfyj/memcached:master-a9a55bf  /bin/bash
    
      
### 方法2

1.获取PID

    PID=$(docker inspect --format " { { .State.Pid }}" <container-id>)

2.连接

    nsenter --target $PID --mount --uts --ipc --net --pid
    
组合起来：

    PID=$(docker inspect --format " { { .State.Pid }}" <container-id>) && nsenter --target $PID --mount --uts --ipc --net --pid
    
创建别名

    alias c='docker ps && read c_id && PID=$(docker inspect --format "{ { .State.Pid }}" $c_id) && nsenter --target $PID --mount --uts --ipc --net --pid'

## 大扫除（清除所有容器 && 镜像）：

      docker kill $(docker ps -q); docker rm $(docker ps -a -q)
      docker rmi $(docker images -q -a) 

# Supervisor


# memcached 扩展


```Shell
cd /tmp && \
git clone https://github.com/php-memcached-dev/php-memcached -b php7 && \ 
cd php-memcached && phpize && \
./configure --with-libmemcached-dir=/usr/local/libmemcached --with-php-config=/usr/local/php/bin/php-config  --disable-memcached-sasl \ 
&& make && make install

```


