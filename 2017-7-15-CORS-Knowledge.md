---
title: CORS 知识点汇总
date:  2017-07-15 10:19:30
tags: [http,html,js,服务端,踩坑]
categories: 踩坑
grammar_cjkRuby: true
---

踩坑集锦

<!-- more -->

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

**注意**：`Access-Control-Allow-Origin` 不能为 `"*"`，否则 cookie 传递失效

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

`Jquey` 中添加 `xhrFields` 参数：

````
$.ajax({
   url: a_cross_domain_url,
   xhrFields: {
      withCredentials: true
   }
});
````

`Zepto`  有 Bug 暂时无解，因为 Zepto 会在`open`之前设置`withCredentials` ，除非修改 ajax 源代码。


### 参考文献
[WHATWG 的 XHR 标准](https://xhr.spec.whatwg.org/#the-withcredentials-attribute)
[CORS 跨域发送 Cookie](http://harttle.com/2016/12/28/cors-with-cookie.html)
[webview的CORS跨域](http://www.iamaddy.net/2014/04/webview%E7%9A%84cors%E8%B7%A8%E5%9F%9F/)
[前方有坑，请绕道——Zepto 中使用 CORS](https://aotu.io/notes/2015/10/26/zepto-cors/index.html)