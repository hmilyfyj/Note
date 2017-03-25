title: Composer 的安装、配置、切换国内源
date: 2016-02-11 08:43
tags: [Composer]
categories: Composer
---


>所有配置都在 Centos7 下通过运行。

<!-- more -->

---

# 安装

```Shell
curl -sS https://getcomposer.org/installer \
        | php -- --install-dir=/usr/local/bin --filename=composer
```

# 获取配置

    sudo composer config -l -g
    
# 切换国内源

    composer config -g repo.packagist composer https://packagist.phpcomposer.com
    
[国内源介绍][3]


# 使用

全局安装 Laravel:

    composer global require "laravel/installer" 
    

组合起来用于配置 Docker 环境

```Shell
   curl -sS https://getcomposer.org/installer \
        | php -- --install-dir=/usr/local/bin --filename=composer && \
        composer config -g repo.packagist composer https://packagist.phpcomposer.com && \
        echo "export PATH=$PATH:/usr/local/php/bin:/root/.composer/vendor/bin" >> ~/.bashrc && \
        composer global require "laravel/installer" 
```


# CI

    composer create-project codeigniter/framework


[官网地址][1]
[相关介绍][2]


  [1]: https://getcomposer.org/doc/00-intro.md#installation-nix
  [3]: http://pkg.phpcomposer.com/
  [2]: http://bbs.csdn.net/topics/390250412