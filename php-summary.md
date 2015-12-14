
title: PHP知识点
date: 2015-12-6 22:27:3
tags: [php,ip]
categories: php
---

### 踩过的坑

#### `Using $this when not in object context in`

 静态方法内无法使用$this,解决方法：

    self::method();

[参考地址](http://blog.csdn.net/yageeart/article/details/6662059)

#### json_decode 时整形溢出，变为科学记数法

solutions:
1. `json_decode($arr, TRUE, 512, JSON_BIGINT_AS_STRING)`
2. `preg_replace('/("\w+"):(\d+)/', '\\1:"\\2"', $resp)`  //适配老版PHP

### 小TIPS

#### 获取物理路径
    realpath();
#### 获取服务端、客户端IP
<!-- more --->
    //Server_ip
    gethostbyname($_SERVER["SERVER_NAME"]); 

    //Client_ip
    function clientIP(){   
      $cIP = getenv('REMOTE_ADDR');   
      $cIP1 = getenv('HTTP_X_FORWARDED_FOR');   
      $cIP2 = getenv('HTTP_CLIENT_IP');   
      $cIP1 ? $cIP = $cIP1 : null;   
      $cIP2 ? $cIP = $cIP2 : null;   
      return $cIP;
     }   

> 客户端IP相关的变量
> 1. $_SERVER['REMOTE_ADDR']; 客户端IP，有可能是用户的IP，也有可能是代理的IP。
> 
> 2. $_SERVER['HTTP_CLIENT_IP']; 代理端的IP，可能存在，可伪造。
> 
> 3. $_SERVER['HTTP_X_FORWARDED_FOR']; 用户是在哪个IP使用的代理，可能存在，可以伪造。
> 
> 服务器端IP相关的变量
> 1. $_SERVER["SERVER_NAME"]，需要使用函数gethostbyname()获得。这个变量无论在服务器端还是客户端均能正确显示。
> 
> 2. $_SERVER["SERVER_ADDR"]，在服务器端测试：127.0.0.1（这个与httpd.conf中BindAddress的设置值相关）。在客户端测试结果正确。



