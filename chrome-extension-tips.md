title: Chrome extension tips
date: 2016-02-08 09:12
tags: [chrome,折腾,js]
categories: chrome
---

# 基本格式

    {
    	"manifest_version": 2,
    	"name": "My Extension",
    	"description": "Extension description",
    	"version": "1.0",
    	"permissions": [
    		"tabs"
    	]
    }

# content_script

content_script是植入型的，它会被植入到符合匹配的网站页面上。**在页面加载完成后执行。**content_script最有用的地方是操作网站页面上的DOM。一切平时做前端的一些操作它都可以做，像什么添加、修改、删除DOM，获取DOM值，监听事件等等，都可以很容易的做到。

## 使用方式：

    {
    	"manifest_version": 2,
    	"name": "My Extension",
    	"description": "Extension description",
    	"version": "1.0",
    
    	"content_scripts": {
    		"js": [
    			"content.js"
    		],
    		"css": ["style.css"],
    		"matches": ["http://*.jgb.cn/*","http://*.amazon.com/*","https://*.amazon.com/*"]
    	}
    }

# background_script

它在chrome扩展启动的时候就启动了，做着它的事，而且等待着你给他的指令。 **它没办法控制页面元素，但可以通过content_script告诉它。** ajax同理，如果要在页面打开时向别的服务器请求数据，这时就可以告诉background_script，让它去请求，然后把返回的数据发送给content_script。这样就不会受到浏览器的安全限制影响。

## 使用方式

    {
    	"manifest_version": 2,
    	"name": "My Extension",
    	"description": "Extension description",
    	"version": "1.0",
    
    	"background": {
    		"scripts": [
    			"background.js"
    		]
    	}
    }

### 跨域

默认情况下Ajax是不允许跨域的，但扩展提供了跨域的配置。

    {
    	"permissions": [
    		"http://www.jgb.cn/" // 允许跨域访问www.jgb.cn
    	]
    }

#### Sending a request from a content script looks like this:

    chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
      console.log(response.farewell);
    });

#### Sending a request from the extension to a content script looks very similar, except that you need to specify which tab to send it to. This example demonstrates sending a message to the content script in the selected tab.

    chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
      chrome.tabs.sendMessage(tabs[0].id, {greeting: "hello"}, function(response) {
        console.log(response.farewell);
      });
    });

#### On the receiving end

    chrome.runtime.onMessage.addListener(
      function(request, sender, sendResponse) {
        console.log(sender.tab ?
                    "from a content script:" + sender.tab.url :
                    "from the extension");
        if (request.greeting == "hello")
          sendResponse({farewell: "goodbye"});
      });

[官方文档](https://developer.chrome.com/extensions/messaging)


# localStorage 

 ## 操作

    // 存储/修改
    localStorage.name = 'only';
    localStorage['name'] = 'only';
    
    // 删除
    localStorage['name'] = null; 删除一个
    localStorage.clear(); 删除所有
    
    // 查
    var name = localStorage.name;
    var name = localStorage['name'];

### Warning

**content_script中的localStorage是存储在对应域名下的，所以别的域名是不能访问的。background_script中的localStorage是存储在chrome扩展下的，所以不管什么域名都可以访问它。这一点很重要，如果没有这个特性，扩展的应用场景就会少很多很多。**



参考资料：

http://fuweiyi.com/chrome%E6%89%A9%E5%B1%95/2014/12/18/a-chrome-extension-01.html

http://blog.csdn.net/talking12391239/article/details/41114693

http://www.cnblogs.com/walkingp/archive/2011/04/04/2003875.html

https://developer.chrome.com/extensions/messaging

http://chajian.baidu.com/developer/extensions/getstarted.html
