title: 在 Codeigniter 中使用 Composer
date: 2016-02-10 15:30
tags: [Composer,PHP,Codeigniter]
categories: PHP
---

# 安装Composer

[官方地址](https://getcomposer.org/doc/00-intro.md#installation-nix)

# 引入项目

## 创建 composer.json
    {
        "require": {
                    "monolog/monolog": "*"
        },
        "minimum-stability": "dev"
    }
## 执行命令

    composer update

## 修改配置文件

### 新版CI

#### 修改config.php

$config['composer_autoload'] = './vendor/autoload.php';

### 老版CI

#### 在library下创建：MY_Composer

    <?php
    /**
     * Description of MY_Composer
     *
     * @author Rana
     */
    class MY_Composer 
    {
        function __construct() 
        {
            include("./vendor/autoload.php");
        }
    }

#### 修改autoload.php文件：

    $autoload['libraries'] = array('MY_Composer');

参考资料：

http://codesamplez.com/development/composer-with-codeigniter
