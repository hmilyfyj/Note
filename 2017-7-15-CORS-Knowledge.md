---
title: CORS 知识点汇总
date:  2017-07-15 10:19:30
tags: Note
categories: Note
grammar_cjkRuby: true
---

### 普通跨域
#### 服务端配置
````
$host   = $referer['host'];
$scheme = $referer['scheme'];
					
$response->header('Access-Control-Allow-Origin', "$scheme://{$host}")
	->header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS')
	->header("Access-Control-Allow-Credentials", "true");
````

### 带 Cookie 跨域
