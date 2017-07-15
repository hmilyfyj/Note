---
title: CORS 知识点汇总
date:  2017-07-15 10:19:30
tags: Note
categories: Note
grammar_cjkRuby: true
---

### 一、普通跨域
#### 服务端配置
````php
$host   = $referer['host'];
$scheme = $referer['scheme'];
					
$response->header('Access-Control-Allow-Origin', "$scheme://{$host}")
	->header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS')
````

### 二、带 Cookie 跨域
#### 服务端额外配置

注意：`Access-Control-Allow-Origin` 不能为 `"*"`

````php
	->header("Access-Control-Allow-Credentials", "true");
````

#### 客户端配置


xhr 原生：
````javascript
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
xhr.open("POST", "url", true);
xhr.send();
````

jquey 中添加 `xhrFields` 参数：

````
$.ajax({
   url: a_cross_domain_url,
   xhrFields: {
      withCredentials: true
   }
});
````

zepto 无解。