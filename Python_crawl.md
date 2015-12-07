title: python 爬取知乎
date: 2015-12-6 22:36
tags: [python,crawl]
categories: python
---
### 前言：

> 为了帮都哥解决python 爬虫时遇到的问题，又复习了下python，并且尝试爬取了知乎页面。 在这里记下遇到的问题。

---

###  Fiddler查看Header
正常登录一次 [知乎](http://www.zhihu.com) 并在Fiddler面板查看详细信息

####  踩点

获取请求内容

    POST http://www.zhihu.com/login/email HTTP/1.1
    Content-Type: application/x-www-form-urlencoded; charset=UTF-8
    Accept: */*
    X-Requested-With: XMLHttpRequest
    Referer: http://www.zhihu.com/#signin
    Accept-Language: zh-CN
    Accept-Encoding: gzip, deflate
    User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR 3.0.30729; .NET CLR 3.5.30729)
    Content-Length: 93
    Host: www.zhihu.com
    
    _xsrf=7150c8e5d9ef17589681c80c276611ea&password=test&captcha=5SAK&remember_me=true&email=test

###  模拟请求
#### 取出隐藏提交内容 xsrt 字段
#### 请求验证码
####  构造post数据
#### 尝试登陆
#### 保存cookie方便下次登陆
