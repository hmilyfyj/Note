title: python 爬取知乎
date: 2015-12-6 22:36
tags: [python,crawl]
categories: python
---
### 前言：

> 为了帮都哥解决python 爬虫时遇到的问题，又复习了下python，并且尝试爬取了知乎页面。 在这里记下遇到的问题。

---

###  踩点

正常登录一次 [知乎](http://www.zhihu.com) 并在Fiddler面板查看详细信息

####  Fiddler查看Header

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

#### 全局cookie

    self.cookie = cookielib.LWPCookieJar()
    #设置cookie处理器
    self.cookieHandler = urllib2.HTTPCookieProcessor(self.cookie)
    #设置登录时用到的opener
    self.opener = urllib2.build_opener(self.cookieHandler,urllib2.HTTPHandler)
    urllib2.install_opener(self.opener)

###  模拟请求

    content = urllib2.urlopen('http://www.zhihu.com/')

#### 取出隐藏字段xsrt 

    xsrf = re.search(r'(?<=name="_xsrf" value=")[^"]*(?="/>)', content).group(0);

#### 请求验证码

    content = urllib2.urlopen('http://www.zhihu.com/')
	
####  构造post数据

    post_data = {'_xsrf': xsrf, 'email': '494046250@qq.com', 'password': '8925159qz', 'remember_me': True,
                             'captcha': captcha}

#### 尝试登陆
#### unzip登陆结果

    if decode and "gzip" in decode:
            try:
                content = zlib.decompress(content, 16 + zlib.MAX_WBITS)
            except zlib.error as error:
                Debug.logger.info('解压出错')
                Debug.logger.info('错误信息:{}'.format(error))

#### 保存cookie方便下次登陆

    zhihu_cookie = os.path.abspath('./') + '/zhihu_cookie_new_try.txt'
    #保存cookie
    self.cookie.save(zhihu_cookie);
	#使用cookie
	self.cookie.load(zhihu_cookie)
